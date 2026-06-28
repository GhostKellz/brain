---
type: entity
title: GhostPSI
created: 2026-06-28
updated: 2026-06-28
tags:
  - linux
  - psi
  - cgroups
  - memory
  - gaming
  - rust
status: developing
entity_type: repository
language: Rust
repo_status: experimental
purpose: Rust-powered Linux pressure governor that keeps workstations responsive
  under memory pressure
related:
  - "[[Rust]]"
  - "[[GhostKellz]]"
  - "[[CK Technology LLC]]"
---

# GhostPSI

A Rust-powered Linux pressure governor that keeps gaming and AI workstations
responsive under memory pressure, using kernel PSI, cgroups v2, zram, and
MGLRU. Authored by [[GhostKellz]] / [[CK Technology LLC]], MIT-licensed.

> [!key-insight]
> Enforcement is OFF by default — GhostPSI runs in dry-run mode, observing
> pressure and ranking offenders before it ever throttles anything.

## Stack

- **Language**: Rust (2024 edition, MSRV 1.85) with Tokio.
- **TUI**: Ratatui.
- **Config**: serde/toml.
- **System**: systemd integration.
- **Workspace**: multi-crate — core, process scanning, zram, cgroup control,
  policy engine, IPC, TUI, and daemon.

## Capabilities

- Real-time PSI monitoring via `/proc/pressure/memory`.
- Reversible cgroup enforcement: `cpu.weight`, `io.weight`, `memory.high`.
- Workload-aware offender ranking across AI assistant, editor, browser, build,
  container, VM, and game classes.
- Protected desktop/system processes that are never throttled.
- Staged escalation ladder: observe -> warn -> throttle -> reclaim -> contain
  -> kill.
- Dry-run mode with enforcement disabled by default.
- Fast CLI plus a native TUI.
- Unix-socket IPC.
- Lightweight daemon: idle under 25 MB, under 0.5% CPU.

## Status

Experimental proof of concept, not production-ready; enforcement is disabled by
default. MIT.

## Relationships

Built on [[Rust]]. Complements zram and MGLRU memory tuning at the kernel level,
acting as the pressure-aware policy layer on top of them.
