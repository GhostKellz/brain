---
type: reference
title: "Btrfs Troubleshooting"
created: 2026-06-21
updated: 2026-06-21
tags:
  - btrfs
  - troubleshooting
  - snapshots
status: seed
related:
  - "[[Btrfs Snapshots]]"
  - "[[Snapper]]"
  - "[[Btrfs Restore From Snapshot]]"
---

# Btrfs Troubleshooting

Common Btrfs and [[Snapper]] failures and their fixes.

## "Cannot delete snapshot X since it is the next to be mounted snapshot"

The btrfs default subvolume points at that snapshot, so it can't be removed.

```bash
sudo btrfs subvolume get-default /              # see current default
sudo btrfs subvolume list / | grep "path @$"    # find @'s ID
sudo btrfs subvolume set-default <ID-of-@> /     # point default at @
sudo snapper delete <X>                          # now succeeds
```

## Snapshots not auto-cleaning

```bash
systemctl status snapper-cleanup.timer
sudo snapper cleanup number
sudo snapper cleanup timeline
```

Check retention limits in `/etc/snapper/configs/root`.

## "snapper list" shows nothing

```bash
ls /etc/snapper/configs/    # config exists?
findmnt /.snapshots         # mounted?
sudo mount -a               # if not
```

## "No space left" but disk shows free

Btrfs metadata is full even though data has room. Rebalance:

```bash
sudo btrfs filesystem usage /          # confirm metadata exhaustion
sudo btrfs balance start -dusage=50 /  # rebalance data chunks
```

## Reclaim space after deleting snapshots

Btrfs doesn't free space immediately:

```bash
sudo btrfs filesystem sync /
sudo fstrim -v /     # SSD only
```

## Filesystem integrity check

```bash
sudo btrfs device stats /     # error counters
sudo btrfs scrub start /      # verify checksums
sudo btrfs scrub status /
```

## Subvolume shows wrong size

CoW sharing breaks naive `du`. Use `compsize`:

```bash
sudo pacman -S compsize
sudo compsize /@
```

## System boots to the wrong snapshot

```bash
cat /boot/loader/entries/<entry>.conf   # must contain rootflags=subvol=@
sudo btrfs subvolume get-default /       # should point at @, not a snapshot
```

See [[Btrfs Subvolume Layout]] for why the explicit `subvol=@` boot flag matters.

## Boot fails after restore

From a live USB:

```bash
mount -o subvolid=5 <root-part> /mnt
ls /mnt/@                 # @ exists?
cat /mnt/@/etc/fstab      # UUIDs correct?
blkid <root-part>         # compare
```

If initramfs is missing, regenerate it (see [[Btrfs Restore From Snapshot]]).

## Quick reference

| Command | Purpose |
|---------|---------|
| `snapper list` | List snapshots |
| `sudo btrfs subvolume list /` | List subvolumes |
| `sudo btrfs filesystem usage /` | Usage breakdown |
| `findmnt -no SOURCE /` | What's mounted as root |
| `sudo btrfs scrub start /` | Integrity check |
