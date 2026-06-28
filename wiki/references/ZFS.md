---
type: reference
title: ZFS
created: 2026-06-28
updated: 2026-06-28
tags:
  - zfs
  - filesystem
  - storage
  - snapshots
  - proxmox
  - backup
  - runbook
status: developing
related:
  - "[[Proxmox]]"
  - "[[Btrfs]]"
  - "[[Linux Administration]]"
  - "[[3-2-1 Backup Strategy]]"
  - "[[Restic Backup]]"
---

> [!key-insight] The CoW filesystem + volume manager on the Proxmox cluster. Pooled storage, checksummed self-healing, cheap snapshots, and `send`/`receive` replication. The btrfs analogue on the Arch daily driver is [[Btrfs]]; Proxmox-specific usage (storage backend, ARC vs guest RAM) lives in [[Proxmox]].

ZFS reference: pool/vdev topology, creating pools and datasets, properties,
snapshots/clones, `send`|`receive` replication, scrub/health, ARC tuning, and
sanoid/syncoid automation. Proxmox-specific notes are cross-linked, not
duplicated.

## Mental model

ZFS is both the filesystem **and** the volume manager — no LVM/mdraid
underneath. The hierarchy:

```
pool  (zpool)            # the top-level storage, made of one or more vdevs
└── vdev                 # a redundancy group (mirror, raidz, single disk)
    └── disks            # the physical members
└── dataset (zfs)        # a filesystem carved from the pool (thin, nestable)
└── zvol                 # a block device carved from the pool (for VMs/iSCSI)
```

> [!key-insight]
> **Redundancy lives at the vdev level, and a pool stripes across its vdevs.**
> Lose a whole vdev → lose the pool. You can't add disks to a raidz vdev to grow
> it (until raidz expansion lands); you grow a pool by adding *more vdevs*. Plan
> topology before you create the pool.

### vdev types

| Type | Redundancy | Use when |
|------|-----------|----------|
| single disk | none | scratch, or a vdev you'll mirror later |
| `mirror` | n-way (survives n-1 failures) | VMs, low-latency random IO, easy expansion |
| `raidz1` | 1 disk | capacity with modest redundancy (3–5 disks) |
| `raidz2` | 2 disks | the safe default for wide arrays (6–10 disks) |
| `raidz3` | 3 disks | very wide / long resilver windows |
| `special` | (mirror it) | dedicated metadata/small-block vdev on SSD |
| `log` (SLOG) | (mirror it) | speeds **sync** writes only (NFS/DBs) |
| `cache` (L2ARC) | none | read cache on SSD when ARC isn't enough |

> [!note] Mirrors resilver fast and let you grow a pool a pair at a time; raidz
> resilvers slowly and is rigid. For a VM host, **mirrors of pairs** usually beat
> raidz on IOPS and operational flexibility.

## Create a pool

```bash
# Always reference disks by stable /dev/disk/by-id paths, NOT /dev/sdX
ls -l /dev/disk/by-id/

# Mirror (two disks)
sudo zpool create -o ashift=12 tank mirror \
  /dev/disk/by-id/ata-DISK1 /dev/disk/by-id/ata-DISK2

# raidz2 (six disks)
sudo zpool create -o ashift=12 tank raidz2 \
  /dev/disk/by-id/ata-DISK{1,2,3,4,5,6}

# Add a mirrored SLOG + an L2ARC cache device later
sudo zpool add tank log mirror /dev/disk/by-id/nvme-SLOG1 /dev/disk/by-id/nvme-SLOG2
sudo zpool add tank cache /dev/disk/by-id/nvme-CACHE1
```

> [!key-insight]
> `ashift=12` (4K sectors) is the right default for virtually all modern drives
> and **cannot be changed after vdev creation**. Getting it wrong (e.g. ashift=9
> on a 4Kn disk) permanently tanks performance. When in doubt, 12.

Recommended pool/dataset defaults set at creation:

```bash
sudo zpool create -o ashift=12 \
  -O compression=zstd -O atime=off -O xattr=sa -O acltype=posixacl \
  -O dnodesize=auto -O normalization=formD \
  tank mirror /dev/disk/by-id/ata-DISK1 /dev/disk/by-id/ata-DISK2
```

(`-o` sets pool properties, `-O` sets inheritable dataset properties.)

## Datasets & properties

Datasets are cheap — make one per workload so you can tune/snapshot/quota them
independently.

```bash
sudo zfs create tank/data
sudo zfs create -o mountpoint=/srv/media tank/media
sudo zfs create -o recordsize=16k tank/postgres    # match the DB page/IO size

# Inspect
zfs list -o name,used,avail,refer,mountpoint
zfs get all tank/data
zfs get compressratio tank
```

Key properties:

| Property | Effect |
|----------|--------|
| `compression=zstd` | transparent compression (use it everywhere; often a net speedup) |
| `atime=off` | don't write access times on reads |
| `recordsize` | max logical block (128k default; lower for DB/VM random IO, raise for big sequential) |
| `quota` / `refquota` | cap dataset (incl. / excl. children + snapshots) |
| `reservation` / `refreservation` | guarantee space |
| `sync=standard` | honor sync writes (don't set `disabled` on data you care about) |
| `sharenfs` / `sharesmb` | native share export |
| `mountpoint` | where it mounts (or `legacy` to manage via fstab) |

```bash
sudo zfs set quota=500G tank/media
sudo zfs set recordsize=64k tank/vms
```

> [!note] For VM/zvol block storage use a `volblocksize` (set at zvol create),
> not `recordsize`. On Proxmox the ZFS storage plugin manages zvols for you —
> see [[Proxmox]].

## Snapshots, clones, rollback

Snapshots are instant, read-only, and near-free (CoW — they cost space only as
the live data diverges).

```bash
sudo zfs snapshot tank/data@2026-06-28          # one dataset
sudo zfs snapshot -r tank@nightly               # recursive: dataset + children
zfs list -t snapshot
zfs list -t snapshot -o name,used,creation tank/data

# Browse a snapshot read-only (no mount needed)
ls /tank/data/.zfs/snapshot/2026-06-28/

# Roll the live dataset back (destroys newer snapshots with -r)
sudo zfs rollback tank/data@2026-06-28

# Clone = writable fork of a snapshot (shares blocks until changed)
sudo zfs clone tank/data@2026-06-28 tank/data-clone

sudo zfs destroy tank/data@2026-06-28           # delete a snapshot
sudo zfs destroy -r tank/old                    # dataset + its snapshots/children
```

> [!warning] `zfs destroy` is immediate and unrecoverable. Always
> `zfs destroy -nv ...` first (dry-run, verbose) to see exactly what it would
> remove — recursive destroys take children and snapshots with them.

## Replication — `send` / `receive`

The killer feature: ship a snapshot (or the delta between two snapshots) to
another pool or host, block-exact and resumable.

```bash
# Full send of one snapshot to another local pool
sudo zfs send tank/data@base | sudo zfs recv backup/data

# Incremental: only the delta between @base and @now
sudo zfs send -i tank/data@base tank/data@now | sudo zfs recv backup/data

# Over SSH to another host (raw -w keeps encryption/compression as-is)
sudo zfs send -RI tank@base tank@now \
  | ssh backuphost "zfs recv -F backup/tank"
```

- `-R` replicate the whole dataset tree with properties/snapshots.
- `-I` send **all** intermediate snapshots between two points; `-i` just the delta.
- `-w` raw send (don't decrypt/recompress — required to replicate encrypted
  datasets without the key).
- Add `-s` on `recv` and use a resume token to restart an interrupted transfer.

> [!key-insight] Replication is not a backup policy by itself — pair it with the
> [[3-2-1 Backup Strategy]] (off-site copy + retention + restore tests). For
> file-level encrypted backups to cloud, [[Restic Backup]] complements ZFS
> replication.

## Health, scrub & disk replacement

```bash
zpool status -v tank            # health, errors, resilver/scrub state
zpool list                      # capacity, frag, health per pool
zpool iostat -v tank 5          # live throughput per vdev
zpool history tank              # every admin action ever taken on the pool

sudo zpool scrub tank           # checksum every block; schedule monthly
sudo zpool scrub -s tank        # stop a running scrub
sudo zpool trim tank            # SSD/NVMe TRIM (or set autotrim=on)
sudo zpool clear tank           # reset error counters after resolving

# Replace a failed disk
sudo zpool offline tank /dev/disk/by-id/ata-DEAD
sudo zpool replace tank /dev/disk/by-id/ata-DEAD /dev/disk/by-id/ata-NEW
# watch resilver
watch zpool status tank
```

> [!key-insight]
> ZFS detects **and self-heals** bit-rot — but only if there's redundancy
> (mirror/raidz) for it to repair from, and only when it reads the bad block.
> That's why a periodic `scrub` matters: it proactively reads everything so rot
> is found and repaired before a second failure makes it unrecoverable.

## ARC tuning

The **A**daptive **R**eplacement **C**ache lives in RAM and defaults to up to
50% of system memory. On a hypervisor that competes with guest RAM.

```bash
# Live stats
arc_summary
cat /proc/spl/kstat/zfs/arcstats | grep -E '^(size|c_max|hits|misses) '

# Cap the ARC (example: 8 GiB)
echo "options zfs zfs_arc_max=8589934592" | sudo tee /etc/modprobe.d/zfs.conf
sudo update-initramfs -u && sudo reboot     # Debian/Proxmox; mkinitcpio -P on Arch
```

> [!key-insight]
> On a VM host, pin `zfs_arc_max` to a deliberate ceiling (rule of thumb ~1–2 GiB
> per 16 GiB of host RAM for a VM-heavy node) so the ARC stays a cache, not a
> memory hog that triggers guest ballooning/OOM. This is the one ZFS knob you
> almost always set on [[Proxmox]].

## Automation — sanoid / syncoid

[`sanoid`](https://github.com/jimsalterjrs/sanoid) takes policy-driven snapshots
+ pruning; `syncoid` wraps `send`/`receive` for easy replication.

```ini
# /etc/sanoid/sanoid.conf
[tank/data]
        use_template = production
        recursive = yes

[template_production]
        hourly = 36
        daily = 30
        monthly = 3
        autosnap = yes
        autoprune = yes
```

```bash
sudo sanoid --cron                              # snapshot + prune per policy (run via timer)
syncoid tank/data backuphost:backup/data        # replicate (handles incremental automatically)
syncoid --recursive tank root@backuphost:backup/tank
```

> [!note] Run `sanoid --cron` from a [[systemd Timers|systemd timer]] (the
> package ships units). Snapshots are the local-rollback tier; `syncoid` is the
> replication tier.

## Import / export & boot

```bash
zpool export tank                       # cleanly detach (before moving disks)
zpool import                            # list importable pools
sudo zpool import tank                  # import by name
sudo zpool import -d /dev/disk/by-id -a # import all, searching by-id
sudo zpool import -f tank               # force (e.g. after a crash/host change)

# Native encryption
sudo zfs create -o encryption=on -o keyformat=passphrase tank/secure
sudo zfs load-key tank/secure && sudo zfs mount tank/secure
```

> [!warning] Keep the pool a notch below full — ZFS performance falls off a cliff
> past ~80–90% capacity because the CoW allocator struggles to find contiguous
> free space. Size with headroom and alert on capacity.

## Notes

- Proxmox as a ZFS consumer (storage backend, zvols, ARC vs guests, PBS): [[Proxmox]].
- The btrfs analogue on the Arch daily driver: [[Btrfs]].
- Snapshots are not backups — pair with [[3-2-1 Backup Strategy]] and
  [[Restic Backup]] for off-site/file-level coverage.
- Tuning knobs interact with [[Sysctl Performance Tuning]] and kernel memory.
