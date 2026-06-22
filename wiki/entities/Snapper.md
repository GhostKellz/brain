---
type: entity
title: "Snapper"
created: 2026-06-21
updated: 2026-06-21
tags:
  - btrfs
  - snapshots
  - backup
  - arch-linux
status: seed
related:
  - "[[Btrfs Snapshots]]"
  - "[[Btrfs Restore From Snapshot]]"
  - "[[Btrfs Troubleshooting]]"
  - "[[systemd Timers]]"
---

# Snapper

**Snapper** is a tool for managing [[Btrfs Snapshots]] — automated timeline
snapshots, cleanup by retention policy, and pre/post snapshot pairs around
package transactions. On Arch it pairs with `snap-pac`, which hooks `pacman` so
every upgrade is bracketed by a snapshot.

## Install + base setup

```bash
sudo pacman -S snapper snap-pac

# A dedicated subvolume holds the snapshots
sudo btrfs subvolume create /.snapshots
sudo chmod 750 /.snapshots
sudo chown :wheel /.snapshots

# Mount it (fstab entry uses subvol=@.snapshots)
# UUID=<root-uuid> /.snapshots btrfs subvol=@.snapshots,noatime,compress=zstd:3 0 0
sudo mount /.snapshots

# Create the root config + first snapshot
sudo snapper -c root create --description "Initial root snapshot"
sudo snapper -c root list
```

A separate config can track `/home`:

```bash
sudo snapper -c home create-config /home
sudo chmod 750 /home/.snapshots
sudo chown :wheel /home/.snapshots
sudo snapper -c home create --description "initial home snapshot"
```

## Automation via timers

Snapper ships [[systemd Timers]] for periodic snapshots and retention cleanup:

```bash
sudo systemctl enable --now snapper-timeline.timer   # periodic snapshots
sudo systemctl enable --now snapper-cleanup.timer    # enforce retention
```

`snap-pac` adds the pre/post-`pacman` snapshots automatically — no timer needed.

## Everyday commands

```bash
snapper list                 # list snapshots (current config)
sudo snapper delete <N>      # delete one
sudo snapper delete <A>-<B>  # delete a range
sudo snapper rollback <N>    # roll back (snapshots current state first)
sudo snapper cleanup number  # manual cleanup pass
sudo snapper cleanup timeline
```

Retention limits live in `/etc/snapper/configs/<config>`.

> [!gap]
> The exact retention numbers and partition UUIDs are host-specific. Generic
> defaults give a roughly 30-day local window; tune per machine.

## Common gotchas

- **"Cannot delete snapshot X — next to be mounted"**: the btrfs default
  subvolume points at it. Reset default to `@`. See [[Btrfs Troubleshooting]].
- **`snapper list` shows nothing**: `/.snapshots` not mounted or a broken config.
- Snapshots **do not include `/boot`** when it is a separate partition —
  regenerate initramfs after a restore (see [[Btrfs Restore From Snapshot]]).

## Related

- [[Btrfs Snapshots]] — the underlying mechanism
- [[Btrfs Restore From Snapshot]] — recovery runbook
- [[3-2-1 Backup Strategy]] — snapper is the local-rollback tier only
