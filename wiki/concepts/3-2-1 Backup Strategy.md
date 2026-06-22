---
type: concept
title: "3-2-1 Backup Strategy"
created: 2026-06-21
updated: 2026-06-21
tags:
  - backup
  - disaster-recovery
  - btrfs
  - restic
status: seed
related:
  - "[[Btrfs Snapshots]]"
  - "[[Restic Backup]]"
  - "[[Snapper]]"
---

# 3-2-1 Backup Strategy

The **3-2-1 rule**: keep **3** copies of data, on **2** different media, with
**1** copy off-site. It is the baseline for surviving both "I broke my system"
and "the building burned down" classes of failure.

> [!key-insight]
> Snapshots are not backups. A local snapshot protects against bad changes; it
> does not survive drive or host loss. The layers below are independent on
> purpose — each covers a failure mode the others don't.

## A layered implementation

A practical Linux desktop/workstation mapping:

```
Live data (btrfs @ / @home)
  │  snapshot (instant, same disk)
  ▼
Snapper timeline + pre/post-pacman   ← bad upgrade? reboot into a snapshot
  │  daily, encrypted, file-level
  ▼
restic → NAS (2nd media, on-site)    ← drive dies? restore files
  │  NAS sync job
  ▼
Cloud object storage (off-site)      ← site loss? restore from cloud
```

| Tier | Tool | Protects against | Scope |
|------|------|------------------|-------|
| Local rollback | [[Snapper]] / [[Btrfs Snapshots]] | bad upgrades, fat-fingered configs | whole subvolume, instant |
| File backup (2nd media) | [[Restic Backup]] → NAS | drive failure, file deletion | `/home`, `/data`, encrypted |
| Off-site | NAS → cloud object storage | fire/theft/site loss | encrypted copy |

## Why each tier is separate

- **Snapshots live on the same disk.** Fast, free, but die with the disk.
- **restic is encrypted + deduplicated** and lands on different physical media
  (a NAS), giving a real second copy with history (e.g. keep 7 daily / 4 weekly
  / 6 monthly).
- **Off-site** is the only thing that survives losing the whole location.

## Validate, don't assume

A backup you've never restored is a hypothesis. Periodically:

- `restic check` / restore a sample (see [[Restic Backup]]).
- Actually boot a snapshot or do a test [[Btrfs Restore From Snapshot]].

> [!gap]
> Specific NAS hosts, bucket names, and cloud providers are infrastructure
> details and live in the private tier.

## Related

- [[Snapper]] — the local-rollback tier
- [[Restic Backup]] — the encrypted file-backup tier
- [[Btrfs Restore From Snapshot]] — exercising the rollback tier
