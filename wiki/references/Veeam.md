---
type: reference
title: "Veeam Backup & Replication"
created: 2026-06-28
updated: 2026-06-28
aliases:
  - "Veeam"
  - "VBR"
  - "Veeam Backup and Replication"
  - "Scale-Out Backup Repository"
  - "SOBR"
  - "Fast Clone"
  - "Backup Copy Job"
  - "GFS"
  - "Instant Recovery"
tags:
  - backup
  - disaster-recovery
  - veeam
  - hyper-v
  - immutability
  - object-storage
status: developing
related:
  - "[[3-2-1 Backup Strategy]]"
  - "[[Hyper-V Administration]]"
  - "[[ReFS]]"
  - "[[Cloud Backup Storage]]"
  - "[[Restic Backup]]"
  - "[[Proxmox]]"
---

# Veeam Backup & Replication

**Veeam Backup & Replication (VBR)** is the production image-level backup platform
for virtualized workloads — full and incremental backups of whole VMs, with a
tiered repository model (**Scale-Out Backup Repository**) that lands recent
restore points on fast local disk and ages older ones off to immutable cloud
object storage. It is the on-prem-to-offsite engine that makes
[[3-2-1 Backup Strategy|3-2-1]] real for a [[Hyper-V Administration|Hyper-V]] fleet.

> [!key-insight]
> Veeam backs up the **whole VM as an image**, not files inside it. One job
> captures the entire virtual disk(s) consistently (via Hyper-V's
> snapshot/RCT plumbing), so a restore brings back a *bootable machine* — not a
> pile of files you have to reinstall an OS around. That image granularity is why
> Veeam is the production DR tool while file-level tools like
> [[Restic Backup|restic]] cover the desktop/`/home` tier.

## Backup model: full + incremental

Veeam's bread and butter is a synthetic-incremental chain:

| Type | What it captures | Cost | Cadence |
|------|------------------|------|---------|
| **Active full** | Entire VM image, read fresh from source | Heavy (full read) | Periodic baseline |
| **Synthetic full** | A new full *assembled from existing increments* on the repo | No source re-read | Weekly/monthly |
| **Incremental** | Only blocks changed since last restore point | Light | Daily/hourly |

On **Hyper-V**, incrementals are driven by **Resilient Change Tracking (RCT)** —
Hyper-V's native changed-block tracking — so Veeam reads only the blocks that
moved since the last run instead of re-scanning the whole disk. The result is a
**forward-incremental** chain: one full + a series of increments, with periodic
synthetic fulls so the chain never grows unbounded and restores stay fast.

> [!key-insight]
> A backup *chain* is only as restorable as its weakest link — a corrupt
> increment can poison everything after it. Veeam's **health check** (re-hash the
> restore points) and **SureBackup** (boot the VM in an isolated sandbox and
> verify it actually starts) turn "I have backups" into "I have *proven*
> backups." A restore point you've never test-booted is a hypothesis, exactly as
> in [[3-2-1 Backup Strategy#Validate, don't assume|the 3-2-1 validate rule]].

> [!note] **v13 dropped reverse-incremental and count-based retention.** You can no
> longer create *reversed-incremental* jobs or set retention to "number of restore
> points" in v13 — both are forward-incremental + **time-based** (days/weeks/months)
> now, paired with GFS/immutability. Modern fast-clone repos make forward chains the
> better design anyway.

## Fast Clone (ReFS / XFS block cloning)

The single biggest performance lever for a Veeam repository is **Fast Clone** —
Veeam's use of the filesystem's **block cloning** (reflink) to build synthetic
fulls and merges by *referencing* existing blocks instead of copying them.

> [!key-insight]
> A synthetic full on a normal filesystem physically re-reads and re-writes every
> block of the chain — slow and space-hungry. On **[[ReFS]]** (Windows) or **XFS**
> (Linux), Fast Clone turns that into a near-instant **metadata** operation: the
> new full just points at blocks that already exist. People see merges up to ~4×
> faster and synthetic fulls that cost almost **no extra space**. This is *the*
> reason to format a Veeam repo as ReFS-64K or reflink-XFS rather than NTFS/ext4.

| Repo filesystem | Fast Clone via | Requirement |
|-----------------|----------------|-------------|
| **[[ReFS]]** (Windows/SMB) | ReFS block cloning | ReFS 3.1+, **64 KB cluster**; SMB needs 3.1.1 + duplicate-extents support |
| **XFS** (Linux) | reflinks | XFS with `reflink=1` + CRC, supported kernel |
| Dedup appliances | vendor reflink | ExaGrid/Quantum DXi etc., if vendor supports it |

Fast Clone also accelerates GFS full creation, SOBR rebalancing/evacuation,
capacity-tier downloads, and `.VBK` exports — not just the nightly merge.

## Scale-Out Backup Repository (SOBR)

A **SOBR** is a single logical repository made of multiple **extents**, organized
into tiers. This is the mechanism that implements 3-2-1 inside one construct:

```
                 Scale-Out Backup Repository
   ┌─────────────────────────────────────────────────────────┐
   │  Performance tier   →   Capacity tier    →   Archive tier │
   │  (fast local disk)      (cloud object)       (cold cloud) │
   │  recent restore pts     offloaded/copied     long-term    │
   └─────────────────────────────────────────────────────────┘
        on-site, hot           OFF-SITE              cheapest
```

| Tier | Backed by | Role | 3-2-1 mapping |
|------|-----------|------|---------------|
| **Performance** | Local disk extents (DAS/SAN/NAS) | Hot recent restore points, fast restores | Copy 1 + 2 (different media on-site) |
| **Capacity** | S3-compatible / Azure Blob object storage | Offload + copy older points off-site | The off-site "1" |
| **Archive** | Cold object (Azure Archive, S3 archive) | Cheapest long-term retention | Long-term off-site |

**Capacity tier** has two policies that often combine:
- **Copy** — write restore points to object storage *as they're created* (a second
  copy immediately, before they age out).
- **Move** — *offload* points older than an operational-restore window off local
  disk to object storage, reclaiming the expensive fast tier.

## Backup Copy Jobs

A **Backup Copy Job (BCJ)** is a *separate* job that copies restore points from a
primary backup to a **second repository** on its own schedule and retention. It is
the textbook way to satisfy the off-site "1" of 3-2-1 — distinct from SOBR
capacity-tier offload (which moves data *within* one repository).

> [!key-insight]
> SOBR offload and a Backup Copy Job are different things and people conflate them.
> **Offload** ages data *within a single SOBR* (performance → capacity). A **Copy
> Job** is an independent job duplicating points to a *different* repository/site
> with its *own* retention — so the off-site copy survives even logical problems
> with the primary. You often run both: a Copy Job to a second site, and that
> target can itself be a SOBR that offloads to cloud.

Two **copy modes**:

| Mode | Behaviour | Best for |
|------|-----------|----------|
| **Immediate** | Copies every restore point as soon as it exists | Reliable-link off-site DR, same-day granularity |
| **Periodic** | Copies on the job's own schedule; missed GFS points created after next run | Slow WAN, dedup/cloud targets, high-frequency primaries, GFS-driven archive |

A Copy Job can target a SOBR: copies land in the performance tier, then sealed/old
chains (including GFS fulls) offload to the capacity tier per policy.

## GFS retention (long-term archival)

**Grandfather-Father-Son (GFS)** keeps weekly / monthly / yearly **archive fulls**
for long-term retention — the layer compliance and "restore from 9 months ago"
needs, sitting on top of the short-term rolling chain.

> [!key-insight]
> GFS does **not** just extend short-term retention. When a full is flagged
> weekly/monthly/yearly it is **sealed** for that period — short-term retention
> can't merge or delete it. So a job keeping "14 points + weekly + monthly" holds
> the active 14-point chain **plus** every still-in-window GFS full. Size the repo
> for that, and pair GFS fulls with **immutability** windows that match (a yearly
> full locked for a year). GFS fulls build via **synthetic** method by default
> (uses Fast Clone); switch to **active full** on dedup appliances, which hate the
> random I/O of synthesis.

## Offsite to the cloud (Azure / Wasabi)

The capacity (and archive) tier is how Veeam ships offsite. Both targets are
attached as object-storage repositories:

- **Azure Blob Storage** — capacity tier; **Azure Archive Storage** for the
  archive tier (a native Azure performance→archive pairing).
- **Wasabi / S3-compatible** — Wasabi Cloud Object Storage as a capacity extent;
  pairs with an S3-compatible archive tier. (S3-compatible providers connect the
  same way restic/rclone do — see [[Cloud Backup Storage]] for the
  endpoint/access-key shape.)

> [!warning]
> Object-storage **access keys / secret keys are SECRETS** — never commit them to
> a note or wiki. Store them in a password manager / secrets vault and reference
> by name only. See [[Cloud Backup Storage]].

> [!note] **Mixing providers in one tier needs "sealed mode."** You can't run, say,
> an Amazon S3 extent and an Azure Blob extent both *active* in the same
> performance tier — to add a second provider you must first put the former extent
> into **sealed mode** (read-only: restores yes, new backups no). The same applies
> to mixing immutable and mutable extents — switch one set to sealed.

## Immutability (ransomware resistance)

The point of the offsite copy in 2026 is that it survives ransomware, and that
means **immutability** — the backup data physically cannot be deleted or altered
until its lock expires (object-lock / WORM on the bucket, hardened-repo on local).

- Enable immutability on **any tier** — performance (local hardened repo or
  immutable object), capacity, and archive.
- Supported on Amazon S3, **S3-compatible (Wasabi)**, Google Cloud, and **Azure**
  object storage — and identically for **Hyper-V** workloads.
- While locked, deletion is blocked from *every* path: manual delete, retention
  expiry, S3-browser tools, even the provider's own support.

> [!key-insight]
> **v13 immutability duration — performance vs capacity tier.** v13 added a "for
> the entire retention period" immutability option that drastically cuts S3 API
> calls and is the recommended setting — **but it currently applies only to the
> performance tier / direct-to-object backups**. For the **capacity tier**
> (Wasabi/Azure offsite), v13 still falls back to "minimum immutability period
> only"; if you configure the entire-retention option on a capacity extent, Veeam
> silently switches it back. Plan capacity-tier retention/cost math around the
> minimum-period behavior until a later v13 update extends it.

## 3-2-1-1-0 — the modern evolution

Veeam popularized extending [[3-2-1 Backup Strategy|3-2-1]] to **3-2-1-1-0**:

- **3** copies of data, on **2** media, **1** off-site — the classic rule.
- **+1** copy that is **offline, air-gapped, or immutable** (the ransomware layer —
  this is what the immutability section above delivers).
- **0** errors — backups **verified** recoverable (SureBackup + health check).

The whole doc maps onto this: SOBR copy/offload + Copy Jobs give 3-2-1, object-lock
immutability gives the extra **1**, and SureBackup gives the **0**.

## Restore options

Image-level backups are only useful if recovery is granular and fast. Veeam's
restore surface is the real product value:

| Restore type | What it does | When |
|--------------|--------------|------|
| **Instant Recovery** | Boots the VM **directly from the backup file** (NFS/iSCSI mount) in minutes, then Storage vMotion / Quick Migration to production | Production VM down — restore RTO in minutes |
| **Full VM restore** | Writes the whole VM back to the hypervisor | Planned full recovery |
| **Guest file recovery** | Mounts the image, restore individual files/folders | "I deleted a file inside the VM" |
| **Application-item (Veeam Explorers)** | App-aware item restore for AD, Exchange, SQL, SharePoint, Oracle, PostgreSQL | Restore one mailbox/object/DB/table |
| **Bare-metal (Veeam Agent)** | Restore a physical machine to dissimilar hardware | Physical server / endpoint |
| **Instant Disk / disk publish** | Surface a single disk from a backup | Data-mining a restore point |

> [!key-insight]
> **Instant Recovery** is the headline: it runs the VM straight off the backup
> repository while the real restore copies in the background, collapsing recovery
> time from "hours of copy" to "minutes to power-on." This only works well when the
> repo is fast — another reason for [[ReFS]]/XFS performance tiers. (v13 retired the
> old *U-AIR* wizard; item-level restore is now the Veeam Explorers directly.)

## Transport modes & proxies

A **backup proxy** is the data mover that reads VM data and ships it to the repo.
How it reads the source is the **transport mode**:

| Mode | Path | Best when |
|------|------|-----------|
| **Direct storage** (Direct SAN / Direct NFS) | Proxy reads VM disks straight off the SAN/NFS, bypassing the host | Shared SAN/NFS; lowest host load, highest throughput |
| **Virtual appliance (Hot-Add)** | Proxy is a VM on the same host; disks are hot-added to it | Virtualized proxy, no SAN access |
| **Network (NBD)** | Reads over the management network via the host stack | Fallback; simplest, slowest |

On **Hyper-V**, the equivalent split is **on-host** vs **off-host** proxy (off-host
uses transportable shadow copies on shared storage to offload the host). Scale
throughput by adding proxies; place a repo's proxy close to its storage.

## v13 notes

VBR **v13** is a major architectural shift: a **Linux-based appliance**, a
**web-first console**, reduced Windows dependency, AI-assisted operations, and
hardened ransomware features. For backup design the practical changes are the new
immutability-duration option above and continued object-storage performance work
(fewer API calls on the performance tier).

**Deprecated / discontinued in v13** (plan migrations now — full removal expected
in v14):
- **Reversed-incremental** mode — use forward-incremental + synthetic/active fulls.
- **"Number of restore points" retention** — use **time-based** retention + GFS.
- **U-AIR** wizard — use the Veeam Explorers / standard restore options directly.

## Where Veeam fits vs the rest of the stack

| Tool | Layer | Granularity | Best for |
|------|-------|-------------|----------|
| **Veeam** | Hypervisor image | Whole VM (bootable) | Production VM DR, [[Hyper-V Administration|Hyper-V]] fleets, offsite to cloud |
| [[Restic Backup]] | File/dir | Files, encrypted+dedup | Linux desktop `/home`, `/data` |
| [[Btrfs Snapshots]] | Filesystem | Subvolume, instant | Local rollback (not a backup) |
| [[Proxmox]] PBS | Hypervisor image | VM/CT | The Proxmox-side analogue to Veeam |

Veeam owns the **production virtualized** tier; the others cover desktop files and
local rollback. Together they are the layered [[3-2-1 Backup Strategy|3-2-1]]
implementation — Veeam's SOBR alone spans local (performance) + offsite-cloud
(capacity/archive) within a single repository.

## Pros / trade-offs

**Pros**
- Image-level, application-consistent VM backups with **Instant Recovery** and
  granular item restore (Veeam Explorers).
- SOBR + Backup Copy Jobs tie on-site fast restore, off-site copy, and immutable
  cloud into one design (3-2-1-1-0).
- **Fast Clone** on [[ReFS]]/XFS makes synthetic fulls near-instant and space-free.
- Strong immutability/ransomware story across local + Wasabi/Azure object storage.
- SureBackup/health-check actually *prove* restorability (the "0" in 3-2-1-1-0).

**Trade-offs**
- Commercial/licensed; heavier to stand up than file-level tools.
- Capacity-tier immutability still on the minimum-period model in v13 — affects
  cloud cost/retention planning.
- Provider-mixing rules (sealed mode) add design constraints to multi-cloud SOBRs.
- Object-storage egress/retrieval costs apply on large cloud restores — size the
  operational-restore window so routine restores stay on the local tier.

## Related

- [[3-2-1 Backup Strategy]] — the rule (and 3-2-1-1-0 evolution) SOBR + Copy Jobs implement
- [[Hyper-V Administration]] — the hypervisor Veeam backs (RCT-driven incrementals)
- [[ReFS]] — the repo filesystem that powers Fast Clone (64K block cloning)
- [[Cloud Backup Storage]] — the S3/Azure/Wasabi target shape + secret handling
- [[Restic Backup]] — the file-level tier alongside Veeam's image tier
- [[Proxmox]] — Proxmox Backup Server, the equivalent on the Proxmox side
