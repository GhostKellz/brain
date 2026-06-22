---
type: concept
title: "Linux Memory Tuning"
created: 2026-06-21
updated: 2026-06-21
tags:
  - linux
  - memory
  - zram
  - performance
status: seed
related:
  - "[[Sysctl Performance Tuning]]"
  - "[[ZRAM Swap]]"
---

# Linux Memory Tuning

Tuning Linux memory behaviour on a RAM-rich workstation is largely about one
philosophy: **fail fast instead of freezing**. When memory is truly exhausted,
you want the kernel to kill one runaway process quickly, not thrash through
swap/compression while the whole machine becomes unresponsive.

> [!key-insight]
> On a high-RAM desktop, *more* swap is not better. Huge swap (or huge ZRAM) lets
> the system grind through gigabytes of pressure before the OOM killer acts —
> producing long freezes. Bounded swap + an early OOM daemon trades a lost
> process for a responsive machine.

## The levers

| Lever | Effect | Tuning intent |
|-------|--------|---------------|
| [[ZRAM Swap]] size | Compressed in-RAM swap | Enough headroom, but bounded so OOM triggers in time |
| `vm.swappiness` | Eagerness to swap anon pages | Lower (e.g. 10) keeps apps in RAM |
| `vm.dirty_ratio` / `dirty_background_ratio` | Dirty-page caps | Lower → earlier writeback, fewer I/O stalls |
| `systemd-oomd` | Userspace OOM daemon | Acts on memory *pressure*, before hard OOM |
| `zswap` | Compressed cache in front of disk swap | Usually disabled when using ZRAM |

These VM tunables are set via [[Sysctl Performance Tuning]].

## systemd-oomd — fail fast

`systemd-oomd` watches PSI (pressure stall information) and kills cgroups under
sustained memory/IO pressure *before* the kernel's last-resort OOM killer would.

```bash
sudo systemctl enable --now systemd-oomd
```

This aligns with the "lose one process, not the whole desktop" goal.

## ZRAM vs zswap

- **ZRAM** is a compressed block device used as swap entirely in RAM.
- **zswap** is a compressed write-back cache in front of *disk* swap.

Running both is redundant; the common choice is ZRAM on, zswap off. Check zswap:

```bash
cat /sys/module/zswap/parameters/enabled    # N when disabled
```

## A sensible workstation profile

- ZRAM: bounded (e.g. a fixed 16 GB) rather than `ram/2` on a 64 GB box.
- `vm.swappiness = 10`, `vm.vfs_cache_pressure = 50`.
- `vm.dirty_ratio = 15`, `vm.dirty_background_ratio = 5`.
- `systemd-oomd` enabled.

> [!gap]
> Exact ZRAM size and ratios depend on total RAM and workload — the above is a
> desktop-oriented starting point, not a universal rule.

## Related

- [[ZRAM Swap]] — the compressed-swap mechanism in detail
- [[Sysctl Performance Tuning]] — where the VM tunables live
