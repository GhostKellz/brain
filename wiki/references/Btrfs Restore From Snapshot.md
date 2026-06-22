---
type: reference
title: "Btrfs Restore From Snapshot"
created: 2026-06-21
updated: 2026-06-21
tags:
  - btrfs
  - recovery
  - snapshots
  - runbook
status: seed
related:
  - "[[Btrfs Snapshots]]"
  - "[[Snapper]]"
  - "[[Btrfs Subvolume Layout]]"
  - "[[Btrfs Troubleshooting]]"
---

# Btrfs Restore From Snapshot

Runbook for restoring an Arch system from a [[Snapper]] snapshot, for a
systemd-boot + NVMe layout. Two paths: from a live USB (system won't boot) or
in-place (`snapper rollback`, system still boots).

## Prerequisites

- Live USB (Arch or CachyOS ISO).
- Your root partition device, e.g. `<root-part>` (`/dev/nvme0n1p2` on this host).
- Target snapshot number from `snapper list`.

> [!gap]
> Device nodes and UUIDs below are host-specific placeholders. Confirm with
> `lsblk` / `blkid` on the actual machine.

## Method 1 — restore from live USB (system won't boot)

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

## Method 2 — rollback in place (system still boots)

```bash
snapper list
sudo snapper rollback <N>   # snapshots current state first, then switches
sudo reboot
```

## Emergency — `@` is completely gone

```bash
mount -o subvolid=5 <root-part> /mnt
btrfs subvolume list /mnt
btrfs subvolume snapshot /mnt/@snapshots/<N>/snapshot /mnt/@

# Make the new @ the default subvolume
btrfs subvolume show /mnt/@          # note the subvolume ID
btrfs subvolume set-default <ID> /mnt
umount /mnt && reboot
```

## Verify after restore

```bash
findmnt -no SOURCE /                 # expect: <root-part>[/@]
sudo btrfs subvolume get-default /   # expect: path @
```

## initramfs missing after restore

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
> boots `@` regardless of the btrfs default subvolume — the most robust setup.
> See [[Btrfs Subvolume Layout]].

## Related

- [[Btrfs Subvolume Layout]] — partition/subvolume reference
- [[Btrfs Troubleshooting]] — boot-to-wrong-snapshot and related fixes
- [[Initramfs FSCK Recovery]] — recovering a non-btrfs root that drops to initramfs
