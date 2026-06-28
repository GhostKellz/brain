---
type: reference
title: "Snapper"
created: 2026-06-21
updated: 2026-06-28
aliases:
  - "snap-pac"
  - "snapper-rollback"
tags:
  - btrfs
  - snapshots
  - backup
  - arch-linux
  - pacman
status: developing
related:
  - "[[Btrfs Snapshots]]"
  - "[[Btrfs]]"
  - "[[systemd Timers]]"
  - "[[3-2-1 Backup Strategy]]"
  - "[[Pacman Hooks]]"
---

# Snapper

**Snapper** manages [[Btrfs Snapshots]]: automated timeline snapshots, retention
cleanup, and pre/post snapshot **pairs** around package transactions. On Arch it
pairs with **`snap-pac`**, which hooks `pacman` so every upgrade is bracketed by a
snapshot you can boot back into when an update breaks something.

> [!key-insight]
> Snapper is the **local-rollback** tier, not a backup. It protects against bad
> upgrades and fat-fingered configs — "boot yesterday's working system" — but it
> lives on the same disk and dies with it. It's the first layer of
> [[3-2-1 Backup Strategy|3-2-1]]; [[Restic Backup|restic]] (off-disk) and an
> off-site copy are the layers that survive drive/host loss.

## Subvolume layout (do this first)

Snapper assumes a flat Btrfs subvolume layout where `/` and `/home` are their own
subvolumes and snapshots live in a dedicated `@snapshots`. Excluding `@pkg`,
`@log`, and friends keeps the package cache and logs *out* of root snapshots, so a
rollback doesn't drag volatile data with it.

```
@           -> /
@home       -> /home
@snapshots  -> /.snapshots
@pkg        -> /var/cache/pacman/pkg
@log        -> /var/log
```

`/etc/fstab` mounts each separately (subvol + zstd compression):

```
UUID=xxx  /                      btrfs  subvol=@,compress=zstd:3,ssd,discard=async,space_cache=v2  0 0
UUID=xxx  /home                  btrfs  subvol=@home,compress=zstd:3,ssd,discard=async,space_cache=v2  0 0
UUID=xxx  /.snapshots            btrfs  subvol=@snapshots,compress=zstd:3,ssd,discard=async,space_cache=v2  0 0
UUID=xxx  /var/cache/pacman/pkg  btrfs  subvol=@pkg,compress=zstd:3,ssd,discard=async,space_cache=v2  0 0
UUID=xxx  /var/log               btrfs  subvol=@log,compress=zstd:3,ssd,discard=async,space_cache=v2  0 0
```

## Install + create the root config

```bash
sudo pacman -S snapper snap-pac btrfs-progs

# Creates /etc/snapper/configs/root and the /.snapshots subvolume
sudo snapper -c root create-config /

# Permissions so the wheel group can read/manage snapshots
sudo chmod a+rx /.snapshots
sudo chown :wheel /.snapshots
```

A separate config can track `/home` the same way:

```bash
sudo snapper -c home create-config /home
sudo chmod 750 /home/.snapshots
sudo chown :wheel /home/.snapshots
```

## The config file (`/etc/snapper/configs/root`)

This is where retention is tuned. The two cleanup algorithms work together:
**number** cleanup prunes the pacman pre/post pairs, **timeline** cleanup prunes
the periodic safety-net snapshots.

```ini
SUBVOLUME="/"
FSTYPE="btrfs"

# fraction of the filesystem snapshots may use / must keep free
SPACE_LIMIT="0.5"
FREE_LIMIT="0.2"

# who may manage this config
ALLOW_USERS="<your-user>"
ALLOW_GROUPS="wheel"
SYNC_ACL="yes"

# number cleanup — prunes pacman (snap-pac) pre/post pairs
NUMBER_CLEANUP="yes"
NUMBER_MIN_AGE="3600"      # don't delete anything younger than 1h
NUMBER_LIMIT="10"          # keep ~10 pacman snapshots (~5 days of updates)
NUMBER_LIMIT_IMPORTANT="2" # keep 2 flagged "important"

# timeline cleanup — periodic safety-net snapshots
TIMELINE_CREATE="yes"
TIMELINE_CLEANUP="yes"
TIMELINE_MIN_AGE="3600"
TIMELINE_LIMIT_HOURLY="0"
TIMELINE_LIMIT_DAILY="7"   # 7 daily — non-pacman changes
TIMELINE_LIMIT_WEEKLY="0"
TIMELINE_LIMIT_MONTHLY="0"
TIMELINE_LIMIT_YEARLY="0"

# drop pre/post pairs where nothing actually changed
EMPTY_PRE_POST_CLEANUP="yes"
EMPTY_PRE_POST_MIN_AGE="3600"
```

> [!key-insight]
> A workstation rarely wants hourly/monthly/yearly snapshots — they bloat the
> store for little benefit. A tight policy of **10 number (pacman) + 7 daily**
> caps the set at ~17 snapshots: enough to undo any recent upgrade or a few days
> of mistakes, without the space creep of a full hourly→yearly timeline. Bump
> `NUMBER_LIMIT`/`TIMELINE_LIMIT_DAILY` only if a machine changes a lot.
> `NUMBER_MIN_AGE`/`TIMELINE_MIN_AGE` (1h) stop cleanup from deleting a snapshot
> you took moments ago.

After editing, no restart is needed — the next timer/cleanup run reads it.

## Pacman hooks: snap-pac

`snap-pac` is the whole reason Snapper shines on Arch. It installs `pacman` hooks
(see [[Pacman Hooks]]) that fire on every transaction:

- A **pre** snapshot *before* packages change.
- A **post** snapshot *after*.

No timer, no config — installing the package is enough. Each upgrade becomes a
labelled pre/post pair, so a bad update is a one-command rollback to the *exact*
pre-upgrade state. `EMPTY_PRE_POST_CLEANUP` discards pairs where nothing on disk
actually differed, keeping the list meaningful.

## Automation: timeline timers

Snapper ships [[systemd Timers]] for periodic snapshots and retention enforcement:

```bash
sudo systemctl enable --now snapper-timeline.timer   # create periodic snapshots
sudo systemctl enable --now snapper-cleanup.timer    # apply NUMBER/TIMELINE limits
```

`snapper-timeline.timer` honors the `TIMELINE_LIMIT_*` knobs (so with only
`DAILY=7`, you get one kept daily snapshot, seven deep). `snapper-cleanup.timer`
runs the number + timeline + empty-pre-post cleanup algorithms.

## Everyday commands

```bash
snapper list                       # list snapshots (root config)
sudo snapper create -d "before X"  # manual labelled snapshot
snapper diff 1..2                  # what changed between two snapshots
sudo snapper -c root cleanup number   # manually trigger number cleanup
sudo snapper delete <N>            # delete one
sudo snapper delete <A>-<B>        # delete a range
```

## Rollback

`snapper rollback` works when booted normally, but the robust path for a broken
system is a manual rollback from a live ISO / recovery shell, swapping the `@`
subvolume for a snapshot:

```bash
# From a live environment, mount the btrfs top-level (subvolid 5)
mount -o subvolid=5 /dev/nvme0n1p2 /mnt

# Move the broken root aside, promote the target snapshot to @
mv /mnt/@ /mnt/@.broken
btrfs subvolume snapshot /mnt/@snapshots/<N>/snapshot /mnt/@

# Reboot — root is the restored @
```

See [[Btrfs#Restore from snapshot]] for the full runbook.

## The default-subvolume gotcha

The Btrfs **default subvolume** must point at `@`, never at a snapshot — otherwise
you boot *into a read-only snapshot* and Snapper can't delete it ("next to be
mounted").

```bash
sudo btrfs subvolume get-default /        # check
sudo btrfs subvolume list /               # find @'s subvolid
sudo btrfs subvolume set-default <id> /   # set default back to @
```

Boot entries should always pin `rootflags=subvol=@` explicitly so the bootloader
doesn't follow a stale default.

## Common gotchas

- **"Cannot delete snapshot — next to be mounted"**: default subvolume points at a
  snapshot. Reset it to `@` (above). See [[Btrfs#Troubleshooting]].
- **`snapper list` shows nothing**: `/.snapshots` not mounted, or a broken config.
- **`/boot` is not included** when it's a separate partition — regenerate
  initramfs after a restore (see [[Btrfs#Restore from snapshot]]).
- **`@home` isn't snapshotted** unless you make a `home` config — root rollback
  won't touch `/home` (usually what you want).

## Related

- [[Btrfs Snapshots]] — the underlying CoW mechanism
- [[Btrfs]] — subvolume layout, restore runbook, troubleshooting
- [[Pacman Hooks]] — how snap-pac wires into pacman transactions
- [[systemd Timers]] — the timeline/cleanup automation
- [[3-2-1 Backup Strategy]] — Snapper is the local-rollback tier only
