---
type: entity
title: "ghostbrew"
created: 2026-06-21
updated: 2026-06-22
tags:
  - scheduler
  - sched-ext
  - ebpf
  - bpf
  - gaming
  - amd
  - zen5
  - rust
  - linux
status: seed
entity_type: repository
language: Rust
repo_status: active
purpose: "sched-ext BPF CPU scheduler for AMD Zen4/Zen5 X3D (V-Cache) and Intel hybrid, tuned for gaming and desktop latency"
related:
  - "[[Rust]]"
  - "[[eBPF and Linux Schedulers]]"
  - "[[CachyOS and TKG Kernels]]"
  - "[[Ghost Ecosystem]]"
  - "[[GhostKellz]]"
---

# ghostbrew

A custom **sched-ext (`scx`) BPF CPU scheduler** (`scx_ghostbrew`) for AMD Zen4/
Zen5 X3D processors and Intel 12th–14th-gen hybrid parts, written in [[Rust]]
with BPF. It combines BORE-style burst detection with hardware-aware scheduling
to favour interactive/gaming workloads. By [[GhostKellz]].

> [!key-insight]
> ghostbrew is **not** a package manager — earlier notes conflated it with the
> Arch package tooling. It is a userspace scheduler loaded via the kernel's
> [[eBPF and Linux Schedulers|sched-ext]] framework: it can be attached and
> detached at runtime with no kernel rebuild, and if it crashes the kernel's
> EEVDF class transparently takes back over.

## How it works

The userspace Rust component handles topology detection, burst tracking, and
V-Cache coordination, then feeds policy to `ghostbrew.bpf.c`, which makes the
actual in-kernel scheduling decisions through sched-ext. EEVDF is the fallback
base when `scx` isn't active.

## What it targets

- **AMD X3D / V-Cache awareness** — detects which CCD carries the 3D V-Cache and
  pins latency-sensitive (game) threads there, while productivity/CPU-hog tasks
  go to the higher-clocking frequency CCD.
- **CCX-local scheduling & NUMA** — keeps tasks on the same CCX to cut L3 misses,
  penalises needless cross-CCD migration, and respects chiplet memory locality.
- **Gaming heuristics** — BORE-inspired burst detection favouring interactive
  over batch work, Wine/Proton process detection, futex/low-latency wakeup for
  audio and input threads, and frame-pacing hints.
- **Runtime tunable** — parameters adjust live through `/run/ghostbrew/control`
  without a restart; integrates with the `linux-ghost` GHOST scheduler kernel
  patch and the `ghost-vcache` mode-switching tool.

## Requirements & usage

Needs a kernel with sched-ext (`CONFIG_SCHED_CLASS_EXT=y`) — e.g. `linux-ghost`,
`linux-cachyos`, or mainline 6.12+. See [[CachyOS and TKG Kernels]] for the
kernels this runs on.

```bash
# start the scheduler (gaming profile prefers the V-Cache CCD)
sudo ghostbrew run --gaming
```

Built on [[Rust]]; part of the [[Ghost Ecosystem]] and a sibling to the gaming/
performance side of the stack rather than the Arch package tooling.
