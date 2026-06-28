---
type: reference
title: Btrfs
aliases:
  - "Btrfs Subvolume Layout"
  - "Btrfs Restore From Snapshot"
  - "Btrfs Troubleshooting"
  - "references/Btrfs-Subvolume-Layout"
  - "references/Btrfs-Restore-From-Snapshot"
  - "references/Btrfs-Troubleshooting"
created: 2026-06-21
updated: 2026-06-28
tags:
  - btrfs
  - filesystem
  - snapshots
  - recovery
  - arch-linux
  - runbook
status: developing
related:
  - "[[Btrfs Snapshots]]"
  - "[[Snapper]]"
  - "[[Initramfs FSCK Recovery]]"
  - "[[ZFS]]"
---

> [!key-insight] The snapshot-friendly filesystem on the Arch daily driver. Flat `@`-prefixed subvolumes + [[Snapper]] give bootable rollbacks; this note is the runbook (layout → restore → troubleshooting). The *concept* (CoW, what a snapshot is) lives in [[Btrfs Snapshots]].

Btrfs reference: subvolume layout, restoring from a snapshot, and
troubleshooting. For the ZFS equivalent on the Proxmox cluster see [[ZFS]].

## Subvolume layout

A flat, `@`-prefixed subvolume scheme is the de-facto standard for snapshot-
friendly Arch installs. Separating volatile/cache directories into their own
subvolumes keeps root snapshots small and meaningful.

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

### Mount options worth setting

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

### Boot entry — pin the subvolume

Make the bootloader mount `@` explicitly so the system never accidentally boots
a snapshot if the btrfs default subvolume drifts:

```
# /boot/loader/entries/<entry>.conf
options root=UUID=<root-uuid> rw rootflags=subvol=@ ...
```

`rootflags=subvol=@` overrides whatever `btrfs subvolume get-default` says — the
single most important line for predictable boots after snapshot operations.

### Inspecting

```bash
sudo btrfs subvolume list /
findmnt -no SOURCE /                 # what's mounted as root
sudo btrfs subvolume get-default /   # default subvolume
```

## Restore from snapshot

Restoring an Arch system from a [[Snapper]] snapshot, for a systemd-boot + NVMe
layout. Two paths: from a live USB (system won't boot) or in-place
(`snapper rollback`, system still boots).

### Prerequisites

- Live USB (Arch or CachyOS ISO).
- Your root partition device, e.g. `<root-part>` (`/dev/nvme0n1p2` on this host).
- Target snapshot number from `snapper list`.

> [!gap]
> Device nodes and UUIDs below are host-specific placeholders. Confirm with
> `lsblk` / `blkid` on the actual machine.

### Method 1 — restore from live USB (system won't boot)

```bash
# 1. Mount the btrfs toplevel (subvolid=5 is always the root of the fs)
mount -o subvolid=5 <root-part> /mnt

# 2. See available snapshots
ls /mnt/@snapshots/

# 3. (optional) keep the broken root aside
mv /mnt/@ /mnt/@.broken

# 4. Recreate @ from a snapshot
btrfs subvolume snapshot /mnt/@snapshots/<N>/snapshot /mnt/@

# 5. After confirming it boots, remove the broken copy
btrfs subvolume delete /mnt/@.broken

# 6. Reboot
umount /mnt && reboot
```

### Method 2 — rollback in place (system still boots)

```bash
snapper list
sudo snapper rollback <N>   # snapshots current state first, then switches
sudo reboot
```

### Emergency — `@` is completely gone

```bash
mount -o subvolid=5 <root-part> /mnt
btrfs subvolume list /mnt
btrfs subvolume snapshot /mnt/@snapshots/<N>/snapshot /mnt/@

# Make the new @ the default subvolume
btrfs subvolume show /mnt/@          # note the subvolume ID
btrfs subvolume set-default <ID> /mnt
umount /mnt && reboot
```

### Verify after restore

```bash
findmnt -no SOURCE /                 # expect: <root-part>[/@]
sudo btrfs subvolume get-default /   # expect: path @
```

### initramfs missing after restore

Snapshots exclude `/boot` (separate partition). Regenerate after chrooting:

```bash
mount <root-part> /mnt -o subvol=@
mount <boot-part> /mnt/boot
arch-chroot /mnt
mkinitcpio -P
exit
```

> [!key-insight]
> An explicit `rootflags=subvol=@` in the boot entry means the system always
> boots `@` regardless of the btrfs default subvolume — the most robust setup
> (see [Boot entry](#boot-entry-pin-the-subvolume) above). For a non-btrfs root
> that drops to initramfs, see [[Initramfs FSCK Recovery]].

## Troubleshooting

### "Cannot delete snapshot X since it is the next to be mounted snapshot"

The btrfs default subvolume points at that snapshot, so it can't be removed.

```bash
sudo btrfs subvolume get-default /              # see current default
sudo btrfs subvolume list / | grep "path @$"    # find @'s ID
sudo btrfs subvolume set-default <ID-of-@> /     # point default at @
sudo snapper delete <X>                          # now succeeds
```

### Snapshots not auto-cleaning

```bash
systemctl status snapper-cleanup.timer
sudo snapper cleanup number
sudo snapper cleanup timeline
```

Check retention limits in `/etc/snapper/configs/root`.

### "snapper list" shows nothing

```bash
ls /etc/snapper/configs/    # config exists?
findmnt /.snapshots         # mounted?
sudo mount -a               # if not
```

### "No space left" but disk shows free

Btrfs metadata is full even though data has room. Rebalance:

```bash
sudo btrfs filesystem usage /          # confirm metadata exhaustion
sudo btrfs balance start -dusage=50 /  # rebalance data chunks
```

### Reclaim space after deleting snapshots

Btrfs doesn't free space immediately:

```bash
sudo btrfs filesystem sync /
sudo fstrim -v /     # SSD only
```

### Filesystem integrity check

```bash
sudo btrfs device stats /     # error counters
sudo btrfs scrub start /      # verify checksums
sudo btrfs scrub status /
```

### Subvolume shows wrong size

CoW sharing breaks naive `du`. Use `compsize`:

```bash
sudo pacman -S compsize
sudo compsize /@
```

### System boots to the wrong snapshot

```bash
cat /boot/loader/entries/<entry>.conf   # must contain rootflags=subvol=@
sudo btrfs subvolume get-default /       # should point at @, not a snapshot
```

### Boot fails after restore

From a live USB:

```bash
mount -o subvolid=5 <root-part> /mnt
ls /mnt/@                 # @ exists?
cat /mnt/@/etc/fstab      # UUIDs correct?
blkid <root-part>         # compare
```

If initramfs is missing, regenerate it (see
[initramfs missing after restore](#initramfs-missing-after-restore)).

### Quick reference

| Command | Purpose |
|---------|---------|
| `snapper list` | List snapshots |
| `sudo btrfs subvolume list /` | List subvolumes |
| `sudo btrfs filesystem usage /` | Usage breakdown |
| `findmnt -no SOURCE /` | What's mounted as root |
| `sudo btrfs scrub start /` | Integrity check |

## Related

- [[Btrfs Snapshots]] — the CoW snapshot concept (what/why)
- [[Snapper]] — manages the `@snapshots` subvolume + automatic timeline snapshots
- [[Initramfs FSCK Recovery]] — recovering a non-btrfs root that drops to initramfs
- [[ZFS]] — the equivalent CoW/snapshot filesystem on the Proxmox cluster
