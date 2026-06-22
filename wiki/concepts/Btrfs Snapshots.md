---
type: concept
title: "Btrfs Snapshots"
created: 2026-06-21
updated: 2026-06-21
tags:
  - btrfs
  - filesystem
  - snapshots
  - arch-linux
status: seed
related:
  - "[[Snapper]]"
  - "[[Btrfs Restore From Snapshot]]"
  - "[[Btrfs Subvolume Layout]]"
  - "[[3-2-1 Backup Strategy]]"
---

# Btrfs Snapshots

A **Btrfs snapshot** is a copy-on-write (CoW) copy of a subvolume. Because Btrfs
only writes blocks that change after the snapshot is taken, creating a snapshot
is near-instant and initially consumes almost no extra space — divergence costs
space only as the live subvolume changes.

> [!key-insight]
> A snapshot is itself a subvolume. This is why rollback on Btrfs is just
> "make `@` point at a snapshot" rather than copying data back.

## Why snapshots matter

- **Instant local rollback** — a bad package upgrade or config change is a reboot
  into a known-good snapshot away, with no restore copy.
- **Pre/post change capture** — taking a snapshot before and after a `pacman`
  transaction (via [[Snapper]] + `snap-pac`) gives a precise diff and an undo
  point per upgrade.
- **Cheap** — CoW means snapshots only grow as the live data diverges.

Snapshots are *not* a backup. They live on the same disk; a drive failure takes
the snapshots with it. Pair them with an off-disk/off-site copy — see
[[3-2-1 Backup Strategy]] and [[Restic Backup]].

## Core commands

```bash
# List all subvolumes (snapshots included)
btrfs subvolume list /

# Create a snapshot of a subvolume
btrfs subvolume snapshot /mnt/data /mnt/data_snap

# Delete a snapshot
btrfs subvolume delete /mnt/data_snap

# Which subvolume is currently mounted as root
findmnt -no SOURCE /        # e.g. /dev/nvme0n1p2[/@]

# What the default subvolume is
btrfs subvolume get-default /
```

## Snapshots vs the default subvolume

The btrfs "default subvolume" is what mounts when no explicit `subvol=` is given.
Snapper rollback manipulates this, which interacts with bootloader config — see
[[Btrfs Subvolume Layout]] for why an explicit `rootflags=subvol=@` in the boot
entry is the safer pattern.

## Space accounting

Btrfs shares data between snapshots via CoW, so per-file `du` lies about real
usage. Use `compsize` for accurate, CoW-aware measurement:

```bash
sudo pacman -S compsize
sudo compsize /@
```

For freeing space after deleting snapshots and for "No space left but disk shows
free" (metadata exhaustion), see [[Btrfs Troubleshooting]].

## Related

- [[Snapper]] — automation layer (timeline timers, cleanup, configs)
- [[Btrfs Restore From Snapshot]] — recovery runbook
- [[Btrfs Subvolume Layout]] — the `@` / `@home` / `@snapshots` scheme
- [[Btrfs Troubleshooting]] — common failures and fixes
