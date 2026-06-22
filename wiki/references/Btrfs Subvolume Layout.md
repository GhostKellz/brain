---
type: reference
title: "Btrfs Subvolume Layout"
created: 2026-06-21
updated: 2026-06-22
tags:
  - btrfs
  - filesystem
  - arch-linux
status: seed
related:
  - "[[Btrfs Snapshots]]"
  - "[[Snapper]]"
  - "[[Btrfs Restore From Snapshot]]"
---

# Btrfs Subvolume Layout

A flat, `@`-prefixed subvolume scheme is the de-facto standard for snapshot-
friendly Arch installs. Separating volatile/cache directories into their own
subvolumes keeps root snapshots small and meaningful.

## The layout

| Mount point | Subvolume | Why separate |
|-------------|-----------|--------------|
| `/` | `@` | The root snapshot target |
| `/home` | `@home` | Snapshot/roll back user data independently |
| `/.snapshots` | `@snapshots` | Holds the snapshots themselves |
| `/var/cache/pacman/pkg` | `@pkg` | Package cache — no point snapshotting it |
| `/var/log` | `@log` | Logs shouldn't roll back with the system |

A typical partition split (systemd-boot, NVMe):

| Partition | Mount | Type |
|-----------|-------|------|
| p1 | `/boot` | vfat (EFI) |
| p2 | `/`, `/home`, `/.snapshots`, ... | btrfs (one fs, many subvolumes) |

> [!key-insight]
> Excluding `@log` and `@pkg` from the root subvolume means a rollback of `@`
> doesn't wipe logs (you can read why it broke) or re-download every package.

## Mount options worth setting

```
noatime                 # don't write access times on every read
compress=zstd:3         # transparent compression, good ratio/CPU balance
ssd                     # SSD-aware allocation (auto-detected, but pin it)
discard=async           # batched TRIM in the background — no fstrim timer needed
space_cache=v2          # faster free-space tracking on large/aged filesystems
```

A full fstab line for the root subvolume then looks like:

```
UUID=<root-uuid>  /  btrfs  subvol=@,noatime,compress=zstd:3,ssd,discard=async,space_cache=v2  0 0
```

> [!key-insight]
> `discard=async` is the modern replacement for a periodic `fstrim.timer` — it
> queues TRIM in the background rather than synchronously on every delete, so you
> get SSD longevity without the latency spikes of `discard` (sync) or the
> all-at-once stall of a weekly fstrim.

## Boot entry — pin the subvolume

Make the bootloader mount `@` explicitly so the system never accidentally boots
a snapshot if the btrfs default subvolume drifts:

```
# /boot/loader/entries/<entry>.conf
options root=UUID=<root-uuid> rw rootflags=subvol=@ ...
```

`rootflags=subvol=@` overrides whatever `btrfs subvolume get-default` says — the
single most important line for predictable boots after snapshot operations.

## Inspecting

```bash
sudo btrfs subvolume list /
findmnt -no SOURCE /                 # what's mounted as root
sudo btrfs subvolume get-default /   # default subvolume
```

## Related

- [[Btrfs Snapshots]] — snapshots are themselves subvolumes
- [[Btrfs Restore From Snapshot]] — uses `subvolid=5` toplevel mounts
- [[Snapper]] — manages the `@snapshots` subvolume
