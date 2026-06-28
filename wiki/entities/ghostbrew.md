---
type: entity
title: ghostbrew
created: 2026-06-21
updated: 2026-06-28
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
status: developing
entity_type: repository
language: Rust
repo_status: experimental
purpose: sched-ext BPF CPU scheduler for AMD Zen5/X3D (V-Cache aware) and Intel
  hybrid, tuned for gaming and desktop latency
related:
  - "[[Rust]]"
  - "[[eBPF and Linux Schedulers]]"
  - "[[CachyOS and TKG Kernels]]"
  - "[[Ghost Ecosystem]]"
  - "[[GhostKellz]]"
---

# ghostbrew

A custom **sched-ext BPF CPU scheduler** optimized for AMD Zen5/X3D (V-Cache
aware) with gaming-focused burst detection. Written in [[Rust]] (2024 edition,
v0.3.x) with BPF (`ghostbrew.bpf.c`). By [[GhostKellz]], MIT-licensed.

> [!key-insight]
> ghostbrew is **not** a package manager. It is a userspace scheduler loaded via
> the kernel's [[eBPF and Linux Schedulers|sched-ext]] framework: it can be
> attached and detached at runtime with no kernel rebuild, and if it crashes the
> kernel's EEVDF class transparently takes over.

## Language & stack

- **Language**: Rust 2024 plus BPF (`ghostbrew.bpf.c`).
- **Tooling**: libbpf-rs / libbpf-cargo; clap; nix/libc; scx_utils / scx_stats;
  criterion for benchmarks.

## How it works

The userspace Rust component handles topology detection, burst tracking, and
V-Cache coordination, then feeds policy to `ghostbrew.bpf.c`, which makes the
in-kernel scheduling decisions through sched-ext. EEVDF is the fallback base when
sched-ext isn't active.

## Capabilities

- **AMD Zen5/X3D**: V-Cache CCD detection with CCD-local/CCX-aware scheduling,
  preferred-core (P-State) handling, and NUMA chiplet optimization.
- **Gaming**: BORE-inspired burst detection, Wine/Proton detection, futex-aware
  prioritization, low-latency audio/input wakeups, and frame-pacing hints.
- **Intel hybrid**: P/E core support for 12th–14th gen parts.
- **Task classification**: Gaming, Interactive, Audio-Input, Productivity, and
  Background classes.
- **Runtime tunable**: parameters adjust live through a control socket without a
  restart; graceful EEVDF fallback.
- **Operations**: benchmarking and support bundles; systemd service.

## Requirements & usage

Needs a kernel with sched-ext (`CONFIG_SCHED_CLASS_EXT=y`) — CachyOS 6.12+ or
mainline 7.x. Ships two binaries: `scx_ghostbrew` (loader) and `ghostbrew`
(CLI). See [[CachyOS and TKG Kernels]] for compatible kernels.

## Status

v0.3.x, experimental MVP / proof of concept. MIT-licensed.

## Relationships

Built on [[Rust]]; part of the [[Ghost Ecosystem]] and the gaming/performance
side of the stack. See [[eBPF and Linux Schedulers]] for background.
