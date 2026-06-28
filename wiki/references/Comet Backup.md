---
type: reference
title: "Comet Backup"
created: 2026-06-28
updated: 2026-06-28
aliases:
  - "Comet"
  - "Comet Server"
  - "Backup Manager"
  - "Storage Vaults"
tags:
  - comet
  - backup
  - msp
  - wasabi
  - storage
status: developing
related:
  - "[[3-2-1 Backup Strategy]]"
  - "[[Veeam]]"
  - "[[Restic Backup]]"
  - "[[Cloud Backup Storage]]"
  - "[[Hyper-V Administration]]"
---

# Comet Backup

**Comet Backup** is a flexible, white-label backup platform built for MSPs and IT
shops: a self-hosted **Comet Server** (the management console + MSP portal) plus a
lightweight **Backup Manager** agent on each endpoint. Its core strength is
**storage-agnostic Storage Vaults** — you point it at your *own* object storage
(here, [[Cloud Backup Storage|Wasabi]] buckets) instead of being locked into a
vendor's hosted tier. It does fast file-level backups *and* image/VM backups,
client-side encrypted and deduplicated.

> [!key-insight]
> Comet's model is **"bring your own storage."** A Storage Vault is just a target
> definition — S3/Wasabi/Azure/B2/local/SFTP — so the data never has to touch
> Comet-the-company. Backups are **chunked, deduplicated, and AES-256 encrypted on
> the client** before upload; the keys are wrapped against the account password, so
> the server (and the storage provider) can't read the data. That's what makes it a
> clean fit for the off-site leg of a [[3-2-1 Backup Strategy]] without trusting a
> middleman.

> [!note]
> I run Comet primarily as the **file-level** backup tool (it also does Hyper-V and
> full image/bare-metal backup), and I use my **own Wasabi buckets** as the Storage
> Vault target rather than Comet-hosted storage — which is Wasabi under the hood
> anyway. Owning the bucket means I control the credentials, lifecycle, and egress.

## Architecture

| Piece | What it is |
|-------|------------|
| **Comet Server** | Self-hosted management console + web portal. Manages users, devices, vaults, policies, jobs, branding. |
| **Backup Manager** | Endpoint agent. `backup-tool` (CLI/service) + `backup-interface` (GUI). Does the chunking/dedup/encryption. |
| **Protected Items** | What to back up (files/folders, Hyper-V, VMware, MSSQL, M365, etc.) — defined per device. |
| **Storage Vaults** | Where backups land — your S3/Wasabi/Azure/B2/local/SFTP. The dedup boundary. |
| **Storage Templates** | Reusable vault definitions that auto-provision a per-user subdirectory/bucket — the MSP scaling lever. |
| **User Policies** | Centrally-enforced retention, schedules, item config, reports — push settings to many users at once. |

## How backups work

Comet uses **variable-sized chunking + client-side deduplication** with an
**incremental-forever** model: after the first backup, only changed chunks upload,
and chunks are deduplicated *across* all Protected Items and snapshots that share a
Vault. Each job finishes with an automated **retention pass** that prunes snapshots
outside the policy.

- **Encryption:** AES-256 (CTR + Poly1305 AEAD). Data keys are encrypted against
  the user's account password — lose the password without a recovery key and the
  data is unrecoverable (by design; zero-knowledge).
- **Dedup boundary = the Vault.** Dedup happens within a Storage Vault, so group
  similar workloads in the same vault to maximize savings.

> [!warning]
> Because dedup/encryption are keyed to the Vault + account, **don't casually
> re-point or recreate a Vault** — you can orphan the dedup chain and force a fresh
> full. Plan vault layout up front.

## Storage Vaults — using your own Wasabi

Two ways the data reaches storage:

1. **Direct-to-cloud (what I use):** the Backup Manager talks straight to the
   **Wasabi** S3 endpoint. With a **Storage Template**, Comet auto-creates a
   per-user **subdirectory (prefix)** inside the bucket and hands each user scoped
   keys — clean multi-tenant separation without standing up a relay.
2. **Self-Hosted Storage Gateway:** Comet Server acts as an in-memory relay in
   front of storage (optionally replicating between back-ends). Useful when you
   want endpoints to never hold cloud creds, at the cost of routing data through
   the server.

```
Endpoint (Backup Manager)
  │  chunk → dedup → AES-256 encrypt  (client-side)
  ▼
Wasabi bucket  (your S3 creds, per-user prefix via Storage Template)
  └── object-locked? → immutable off-site copy
```

> [!key-insight]
> Pair Comet with **Wasabi Object Lock** for immutable, ransomware-resistant
> off-site backups — the same "immutability" win [[Veeam]] gets from SOBR capacity
> tier, but for the file-level/endpoint fleet. This is the **1** (off-site) leg of
> 3-2-1 done with storage you own.

## Workloads

Comet is broad for a lightweight agent:

- **Files & folders** (my main use) — fast incremental file-level.
- **Disk Image / bare-metal** — full system image restore.
- **Hyper-V & VMware** — VM-level backup (see below).
- **Microsoft 365 / Exchange** — cloud-to-cloud.
- **Databases** — MSSQL, MySQL, MongoDB.
- OS coverage: Windows, Linux, macOS.

### Hyper-V backups

Comet backs up [[Hyper-V Administration|Hyper-V]] VMs using **production
checkpoints** (guest VSS for app-consistent state) with fallback to standard
checkpoints. VM data is deduplicated **across VMs and across snapshots** in the
Vault. Restore is flexible: back to **original VHDX format**, **granular file
restore** out of the VM image, or to a **new VM**.

> [!note]
> Comet's Hyper-V/image support overlaps [[Veeam]], but I lean on Veeam for the
> heavy production VM tier (SOBR, RCT incrementals, Instant Recovery) and Comet for
> file-level + lighter endpoint/VM coverage. They're complementary, not redundant.

## Restore

- File/folder granular restore, including browsing into image/VM snapshots.
- Full disk-image / bare-metal restore.
- VM restore to original format, granular files, or a new VM.
- Restores pull only the chunks needed and decrypt client-side.

## MSP portal & multi-tenant ops

The **Comet Server** console is the white-label MSP control plane:

- **Branding / white-label** — rebrand the server, agent, and emails as your own.
- **Storage Templates** — onboard a new client/user and auto-provision their vault
  prefix; the scaling primitive for fleets.
- **User Policies** — enforce retention, schedules, Protected Items, and email
  reports centrally; users can't drift off-policy.
- **Reporting / alerting** — per-job status, email summaries, API for automation.
- Recent releases: **26.4.0 "Phoebe"** (Proxmox guest quotas), **25.9.2** (Wasabi
  US West 2 region, shared quotas).

## Pros / trade-offs

**Pros**
- Bring-your-own-storage (Wasabi/S3/Azure/B2/SFTP/local) — no vendor lock-in.
- Client-side AES-256 + dedup + incremental-forever; zero-knowledge.
- Genuinely white-label MSP portal with Storage Templates + User Policies.
- One agent covers files, image, Hyper-V/VMware, M365, databases.

**Trade-offs**
- Zero-knowledge means **password/recovery-key loss = data loss** — manage keys.
- Dedup is per-Vault; bad vault layout wastes space or breaks the dedup chain.
- Heavy production VM features (Instant Recovery, SOBR tiers) are [[Veeam]]'s lane.
- Self-hosted server is one more thing to patch/back up (back up the Comet Server
  config itself).

## Related

- [[3-2-1 Backup Strategy]] — Comet → Wasabi is the encrypted off-site (the **1**)
- [[Cloud Backup Storage]] — Wasabi as the object-storage target (own buckets)
- [[Veeam]] — the heavy production VM tier; Comet complements it for files/endpoints
- [[Restic Backup]] — the Linux CLI analogue (client-side dedup + encrypted repos)
- [[Hyper-V Administration]] — Comet backs up Hyper-V via production checkpoints
