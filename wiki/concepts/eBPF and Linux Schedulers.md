---
type: concept
title: "eBPF and Linux Schedulers"
created: 2026-06-22
updated: 2026-06-22
tags:
  - ebpf
  - bpf
  - sched-ext
  - scheduler
  - kernel
  - gaming
  - performance
status: developing
related:
  - "[[ghostbrew]]"
  - "[[CachyOS and TKG Kernels]]"
  - "[[Linux and Systems]]"
  - "[[GhostKellz]]"
---

> [!key-insight] sched-ext (`scx`) lets you load a CPU scheduler as a BPF program
> at runtime — swap scheduling policy without a kernel rebuild, and fall back to
> the in-tree scheduler instantly if it misbehaves.

**sched-ext** (extensible scheduler class, `SCHED_EXT`) is a kernel framework,
merged in **Linux 6.12**, that exposes the CPU scheduler as a set of callbacks
implemented in **eBPF**. A userspace program loads a `.bpf.o` that decides which
task runs on which CPU and for how long; the kernel keeps its normal scheduler
(EEVDF) as the safety net.

## Why it matters

- **Hot-swappable policy.** Attach/detach a scheduler live (`scx_<name>`), no
  reboot, no recompiled kernel. Experiment on a running desktop.
- **Safe by construction.** The BPF verifier bounds the program, and a watchdog
  reverts to EEVDF if the scx scheduler stalls or crashes — so a bad scheduler
  can't hard-lock the box.
- **Workload-shaped scheduling.** Schedulers can be tuned for one goal (gaming
  latency, throughput, cloud tail-latency) instead of the one-size-fits-all
  default.

## Requirements

```bash
# kernel must expose the extensible scheduler class
zcat /proc/config.gz | grep SCHED_CLASS_EXT
# CONFIG_SCHED_CLASS_EXT=y
```

Shipped enabled on `linux-cachyos`, `linux-ghost` (TKG), and CachyOS/Zen-derived
kernels; mainline 6.12+ also supports it. See [[CachyOS and TKG Kernels]].

## The scheduler landscape

| Scheduler | Focus |
|-----------|-------|
| **EEVDF** | In-tree default since 6.6 (replaced CFS); the fallback base for scx. |
| **BORE** | Burst-Oriented Response Enhancer — CFS/EEVDF patch favouring interactive tasks; ships on CachyOS/TKG. |
| `scx_lavd` | Latency-criticality aware (Valve/CachyOS) — gaming/handheld (Steam Deck) tuned. |
| `scx_bpfland` | Interactive-first vruntime scheduler. |
| `scx_rusty` | Multi-domain, mostly-userspace Rust scheduler (load balancing). |
| `scx_layered` | Configurable layered policy for datacenter workloads. |
| **[[ghostbrew]]** | Custom scx scheduler for AMD Zen5/X3D V-Cache + gaming, by [[GhostKellz]]. |

## BORE vs sched-ext

They're complementary: **BORE** is a tweak to the in-tree EEVDF scheduler (the
default run path), while **sched-ext** *replaces* the run path with a BPF
program. A CachyOS/TKG kernel can ship BORE as its baseline *and* expose
sched-ext so you can load something like [[ghostbrew]] on top when you want
X3D/gaming-specific behaviour, dropping back to BORE/EEVDF otherwise.

## Related

- [[ghostbrew]] — the custom scx scheduler this vault tracks.
- [[CachyOS and TKG Kernels]] — the kernels that enable sched-ext here.
