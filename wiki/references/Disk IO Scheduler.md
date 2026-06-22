---
type: reference
title: "Disk IO Scheduler"
created: 2026-06-21
updated: 2026-06-21
tags:
  - linux
  - storage
  - performance
  - udev
status: seed
related:
  - "[[Sysctl Performance Tuning]]"
  - "[[Linux Memory Tuning]]"
---

# Disk IO Scheduler

The Linux block layer uses an **I/O scheduler** to decide the order requests hit
a device. The right choice depends on the device type — fast NVMe wants minimal
scheduling, spinning disks benefit from fair queueing.

## Picking a scheduler

| Scheduler | Best for | Notes |
|-----------|----------|-------|
| `none` | NVMe SSDs | Minimal overhead; device handles its own queueing |
| `mq-deadline` | SATA SSDs, modern drives | Balanced latency/throughput |
| `bfq` | HDDs, busy interactive desktops | Fair queueing, good responsiveness |

> [!key-insight]
> On NVMe, the device's own controller queues better than the kernel can, so
> `none` (no extra scheduling) is usually fastest. Adding a scheduler there just
> adds overhead.

## Check current scheduler

```bash
cat /sys/block/<device>/queue/scheduler   # e.g. nvme0n1, sda
# the bracketed entry is active: [none] mq-deadline ...
```

## Set persistently via udev

```bash
# /etc/udev/rules.d/60-ioschedulers.rules
ACTION=="add|change", KERNEL=="nvme[0-9]*", ATTR{queue/scheduler}="none"
ACTION=="add|change", KERNEL=="sd[a-z]",   ATTR{queue/scheduler}="mq-deadline"
```

Reload:

```bash
sudo udevadm control --reload && sudo udevadm trigger
```

## Readahead

How much data the kernel pre-loads on sequential reads. Bigger helps HDDs with
large files; little benefit on NVMe.

```bash
blockdev --getra /dev/<device>          # current (in 512-byte sectors)
sudo blockdev --setra 4096 /dev/<device>
```

## Mount options for responsiveness

- `noatime` — skip writing access times on every read.
- `commit=60` (ext4) — flush the journal every 60s (trades a little safety for
  fewer writes).

## Benchmarking

- `fio` — flexible I/O stress testing
- `iostat` — live I/O usage
- `hdparm` — quick throughput probe

## Related

- [[Sysctl Performance Tuning]] — dirty-ratio writeback tuning
- [[Linux Memory Tuning]] — page cache interacts with I/O
