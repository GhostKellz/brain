---
type: reference
title: "Proxmox Backup Server"
created: 2026-06-28
updated: 2026-06-28
aliases:
  - "PBS"
  - "proxmox-backup-client"
  - "proxmox-backup-manager"
  - "PBS S3"
tags:
  - proxmox
  - backup
  - deduplication
  - s3
  - object-storage
  - disaster-recovery
status: developing
related:
  - "[[Proxmox]]"
  - "[[Veeam|Veeam Backup & Replication]]"
  - "[[3-2-1 Backup Strategy]]"
  - "[[Cloud Backup Storage]]"
  - "[[ZFS]]"
  - "[[Restic Backup]]"
---

> [!key-insight] Generalized from field notes; host/IP/datastore values are placeholders.

**Proxmox Backup Server (PBS)** is the dedicated backup appliance for the Proxmox
ecosystem — the Proxmox-side analogue to [[Veeam|Veeam Backup & Replication]]. It
does **incremental, globally-deduplicated, client-encrypted, verifiable** backups
of VMs, containers, and file trees, replacing plain `vzdump` tarballs. As of
**PBS 4.2 (April 2026)** it can also persist a datastore to **S3-compatible
object storage** (Wasabi, MinIO, Ceph RGW, AWS S3, …).

> [!key-insight]
> PBS is a **content-addressable chunk store**. A backup is split into
> variable-size chunks, each named by its SHA-256 hash; identical chunks are
> stored **once per datastore**. After the first full, the QEMU **dirty-bitmap**
> means later runs only read+ship *changed* blocks — so backups are fast and 20
> near-identical VMs cost roughly one VM of space. The flip side: a corrupt chunk
> poisons every snapshot that references it, which is why **verify** jobs are
> non-negotiable.

## Architecture

| Piece | What it is |
|-------|-----------|
| **Proxmox Backup Server** | The appliance (Debian-based). Owns datastores, runs prune/GC/verify/sync, serves the web UI + API. |
| **Datastore** | A chunk store on a directory (ideally a ZFS dataset) **or** an S3 bucket (4.2+). The dedup/encryption boundary. |
| **Namespace** | A logical subtree inside a datastore (`ns/...`) — lets many PVE hosts/tenants share one datastore without colliding. |
| **`proxmox-backup-client`** | Standalone client for backing up **file/block** data from any Linux host (not just PVE guests). |
| **PVE integration** | A PVE node adds the datastore as **storage**; `vzdump`/the GUI then back guests straight to PBS. |
| **Remote + Sync job** | Pull (or push, 4.x) snapshots between two PBS instances for offsite copies. |

## How a backup actually works

1. **Chunking.** The source stream (a VM disk image, or a `.pxar` file archive)
   is cut into **variable-length chunks** by a rolling hash. Each chunk is
   compressed (zstd) and, if enabled, **encrypted client-side**.
2. **Content addressing.** Each chunk is keyed by its **SHA-256**. Before
   uploading, the client asks PBS "do you already have hash X?" — if yes, it's
   skipped. This is the dedup: globally across every snapshot in the datastore.
3. **Index files.** A snapshot is just an **index** (a `.fidx`/`.didx`) listing
   the ordered chunk hashes that reconstitute each archive, plus a manifest.
4. **Dirty-bitmap incrementals.** For running VMs, QEMU tracks changed blocks in
   a **dirty bitmap**; the next backup only *reads* dirty blocks, then dedups
   them. First backup = full read; every subsequent = changed-blocks-only, yet
   each snapshot is independently restorable (no chain to replay).

```
VM disk ──chunk──► [zstd] ──[encrypt]──► SHA-256 ──► dedup check ──► store chunk
                                                          │ (already present)
                                                          └─► skip, just index
snapshot = manifest + index(ordered chunk hashes)
```

## Standalone client (any Linux host)

`proxmox-backup-client` backs up file trees / block devices from machines that
aren't PVE guests:

```bash
export PBS_REPOSITORY="<user>@pbs@<pbs-host>:<datastore>"
export PBS_PASSWORD="..."                 # or use a token / keyring

# File-level archive of / into a namespace
proxmox-backup-client backup root.pxar:/ --ns <namespace>

# Client-side encryption (key never leaves the host)
proxmox-backup-client backup root.pxar:/ --keyfile /etc/pbs/backup.key

proxmox-backup-client snapshot list
proxmox-backup-client restore <snapshot> root.pxar /restore/target
proxmox-backup-client mount   <snapshot> root.pxar /mnt/restore   # browse via FUSE
```

> [!key-insight]
> **Client-side encryption is zero-knowledge.** With `--keyfile`, chunks are
> AES-256-GCM encrypted *before* they reach PBS, so the server (and any S3 bucket
> behind it) only ever sees ciphertext. **Lose the key = lose the backups** — PBS
> cannot recover it. Escrow the key (`proxmox-backup-client key ...`) to a
> password manager / vault, exactly like the secret-handling rule in
> [[Cloud Backup Storage]].

## PVE integration

```bash
# On the PVE node: add the datastore as storage (GUI: Datacenter → Storage → Add → PBS)
# Then back up a guest:
vzdump <vmid> --storage <pbs-storage> --mode snapshot

# Live-restore: boot the VM immediately while blocks stream in the background
qmrestore <pbs-storage>:backup/vm/<vmid>/<timestamp> <newid> --live-restore

# Single-file restore from a VM backup (GUI: datastore → snapshot → File Restore)
```

## S3 object-storage backend (PBS 4.2+)

Since **PBS 4.0** (July 2025, tech preview) and **GA in 4.2** (April 2026), a
datastore can live on **S3-compatible object storage** instead of (or alongside)
local disk. This replaces the old unofficial hacks — `s3fs-fuse` mounts and
`pmoxs3backuproxy` — that Proxmox never supported.

### How PBS uses the bucket

PBS does **not** drop whole self-contained snapshots into the bucket. It uses the
**same content-addressable chunk store**: every dedup'd/compressed/encrypted
chunk becomes an **individual S3 object**, prefixed by the datastore name; index
files land as objects too. All the core features survive — **dedup, compression,
encryption, prune, verify, GC** — just with S3 as the backing block device.

```
PBS datastore (S3 mode)
├── local cache  ← LRU: hot chunks + ALL logical metadata (MANDATORY, persistent)
└── S3 bucket
    └── <datastore-name>/
        ├── .chunks/<hash>   ← one object per chunk (dedup'd)
        └── <index/manifest objects>
```

### The mandatory local cache

Even though data lives in S3, a **persistent local cache is required** — you give
a filesystem path at datastore-creation time:

- It's an **LRU cache** holding recently-used chunks **plus all logical
  metadata** needed to operate the datastore. The metadata is *why* it must be
  persistent — PBS relies on it; a memory-only cache is **not** supported.
- Size it **64–128 GiB**, on a **dedicated disk/partition or a ZFS dataset with a
  quota**. You **cannot** reuse an existing regular datastore as the cache.
- Purpose: cut S3 API request volume and hide object-store latency.

### Setup

```text
# 1. In your S3 provider FIRST: create the bucket + an access key.
#    PBS does NOT create buckets or manage ACLs.
#    Key needs: GetObject, PutObject, ListBucket, DeleteObject
# 2. PBS GUI: Configuration → Remotes → S3 Endpoints → add endpoint
#    (URL/region, access key, secret key)
# 3. Datastore → Add → backing type S3:
#    pick the endpoint + bucket, give the local cache path + size cap.
```

> [!key-insight]
> **Multiple datastores can share one bucket** — objects are prefixed by the
> datastore name. And for **disaster recovery**: if the PBS host dies, the
> bucket's contents are reusable — but you **cannot restore directly from the
> bucket**. A PBS instance is *always* required to read it, and each S3 datastore
> is exclusively owned by **one** PBS instance at a time (no concurrent
> multi-PBS access to the same datastore).

> [!warning]
> **Do NOT enable S3 Object Lock or versioning on a PBS bucket.** PBS does **not**
> support them, and turning Object Lock on **can corrupt the datastore
> structure**. This is the critical contrast with [[Veeam]] /
> [[Cloud Backup Storage|Wasabi immutability]]: Veeam *wants* Object Lock for ransomware-proof
> immutability; PBS S3 is **incompatible** with it. If you need immutable offsite
> for the Proxmox side, keep a **local PBS** as the verified tier and treat S3 as
> a complementary copy, not the immutable anchor.

> [!warning]
> **Garbage collection on S3 is API-expensive.** GC issues *far* more requests
> than on local storage (it must enumerate/sweep objects). **Schedule it weekly,
> not daily**, and watch for per-request billing — on metered providers the GC
> sweep, not the storage, can dominate cost. PBS 4.2 surfaces per-datastore
> **request counters + traffic stats** in the summary so you can alert on this.

### Wasabi specifics

Wasabi is flat-rate storage with **no egress/API fees** on standard plans, which
neutralizes PBS S3's biggest cost gotcha (the chatty GC). The catch: Wasabi's own
**minimum-storage-duration** still applies, and you must **not** turn on its
object-lock/compliance features for a PBS datastore. Pair `<bucket>` per
environment, prefix-share if you run several datastores, and keep the LRU cache on
local NVMe. See [[Cloud Backup Storage]] for the generic S3 target shape.

## Maintenance — prune, GC, verify

Three independent hygiene jobs (set schedules in the GUI, or run by hand/API):

```bash
# PRUNE — apply a retention policy (keeps last/hourly/daily/weekly/monthly/yearly).
#   Removes snapshot *index entries*; does NOT free space yet.
proxmox-backup-manager prune-job ...        # or per-datastore schedule in GUI

# GARBAGE-COLLECT — sweep chunks no index references anymore; THIS frees space.
proxmox-backup-manager garbage-collection start <datastore>

# VERIFY — re-hash stored chunks to catch bit-rot before a restore needs them.
proxmox-backup-manager verify <datastore>
```

> [!key-insight]
> **Prune ≠ delete space.** Prune only drops the snapshot's *reference*; space is
> reclaimed by **GC**, which deletes chunks nothing points to anymore. Order of
> operations: prune (retention) → GC (reclaim) → verify (integrity). Because dedup
> shares chunks, verify is the safety net — schedule it so rot surfaces on a quiet
> night, not at 2 a.m. during a real restore.

## Sync jobs / offsite copies

A **Remote** + **Sync job** replicates snapshots between two PBS instances for the
offsite leg of [[3-2-1 Backup Strategy|3-2-1]]:

- **Pull** (classic): the destination PBS pulls from a source remote.
- **Push** (4.x): the source pushes to a remote destination.
- **Parallel sync** (4.2): sync jobs run multiple streams concurrently — much
  faster catch-up over WAN links.

This gives you **two PBS copies** (e.g. on-site fast restore + off-site DR) on top
of, or instead of, the S3 tier.

## Restore options

| Method | Use |
|--------|-----|
| **Full VM/CT restore** | `qmrestore` / `pct restore` — rebuild a guest from a snapshot. |
| **Live restore** | Boot the VM immediately, stream blocks in the background — minimal RTO. |
| **File restore** | Browse a VM/CT backup and pull individual files (GUI → File Restore). |
| **`proxmox-backup-client mount`** | FUSE-mount a `.pxar` archive to cherry-pick files. |
| **Disaster recovery (S3)** | Stand up a fresh PBS, attach the bucket as a datastore, restore — bucket alone is not directly restorable. |

## Where PBS fits vs the rest of the stack

| Tool | Layer | Granularity | Best for |
|------|-------|-------------|----------|
| **PBS** | Hypervisor image + files | VM/CT, `.pxar` file trees | The Proxmox-side backup tier; dedup + S3 offsite |
| [[Veeam]] | Hypervisor image | Whole VM (bootable) | Hyper-V/VMware fleets, **Object-Lock-immutable** cloud |
| [[Restic Backup]] | File/dir | Files, encrypted+dedup | Linux `/home`, `/data` from non-PVE hosts |
| [[ZFS]] snapshots | Filesystem | Dataset, instant | Local rollback (**not** a backup) |

> [!key-insight]
> PBS and Veeam solve the same shape on different hypervisors. The one place they
> *diverge sharply* is cloud immutability: **Veeam leans on S3 Object Lock**;
> **PBS forbids it**. So a hardened Proxmox 3-2-1-1-0 keeps the immutable/verified
> copy on a **local PBS with verify jobs** (and optionally an air-gapped/second
> PBS via sync), using **S3 as cheap capacity**, not as the immutable anchor.

## Pros / trade-offs

**Pros**
- Global dedup + dirty-bitmap incrementals → small, fast, independently-restorable backups.
- Client-side zero-knowledge encryption; the server/S3 only sees ciphertext.
- Verify jobs *prove* restorability against bit-rot (the dedup safety net).
- Live restore = low RTO; file-level restore from image backups.
- S3 backend (4.2) makes PBS a near-**stateless** service — data in object storage, only the LRU cache on the box.
- Sync jobs (parallel in 4.2) give clean two-site DR.

**Trade-offs**
- **No S3 Object Lock/versioning** — weaker native immutability than Veeam+Wasabi; enabling Object Lock can corrupt the store.
- S3 **GC is API-heavy** → schedule weekly, watch per-request cost (or use flat-rate Wasabi).
- S3 mode still needs a **mandatory local cache** (dedicated 64–128 GiB) — not truly diskless.
- A PBS instance is **always** required to restore S3 data; the bucket isn't self-serve.
- Each S3 datastore is owned by **one** PBS at a time (no concurrent multi-PBS).

## Related

- [[Proxmox]] — the hypervisor PBS backs; this page expands the PBS section there
- [[Veeam]] — the cross-platform analogue; note the Object-Lock divergence
- [[3-2-1 Backup Strategy]] — sync jobs + S3 implement the offsite/copy legs
- [[Cloud Backup Storage]] — the S3/Wasabi target shape + secret handling
- [[ZFS]] — the recommended datastore filesystem (and the LRU-cache dataset)
- [[Restic Backup]] — the file-level tier for non-PVE Linux hosts

Sources: [Proxmox Backup 4.2 docs — Backup Storage](https://pbs.proxmox.com/docs/storage.html) · [PBS 4.0 release](https://www.proxmox.com/en/about/company-details/press-releases/proxmox-backup-server-4-0) · [PBS 4.2: S3 storage + parallel sync](https://www.helpnetsecurity.com/2026/04/30/proxmox-backup-server-4-2-released/) · [PBS 4.2 S3 exits tech preview (wz-it)](https://wz-it.com/en/blog/proxmox-backup-server-4-2-s3-storage-stable/)
