---
type: entity
title: "Bolt"
created: 2026-06-21
updated: 2026-06-21
tags:
  - containers
  - runtime
  - rust
  - nvidia
  - gpu
status: seed
entity_type: repository
language: Rust
repo_status: active
purpose: "Next-generation container runtime and orchestration platform with built-in NVIDIA GPU passthrough"
related:
  - "[[Rust]]"
  - "[[GhostForge]]"
  - "[[GhostPanel]]"
  - "[[Ghost Ecosystem]]"
  - "[[GhostKellz]]"
---

# Bolt

A next-generation container runtime and orchestration platform, written in
[[Rust]] (configured via a `Boltfile.toml`). By [[GhostKellz]].

Bolt is the container substrate that other ghost\* tools build on:

- [[GhostForge]] uses Bolt as its gaming container runtime.
- [[GhostPanel]] is "Portainer for Bolt" — a web UI for managing Bolt
  environments.

## Built-in GPU passthrough (nvbind)

NVIDIA GPU passthrough is **native to Bolt** — the former standalone `nvbind`
tool is now a built-in runtime path (`src/runtime/nvbind.rs`), not an external
dependency. The Boltfile selects it with `runtime = "nvbind"`, and the CLI
exposes it directly (e.g. `bolt gaming gpu nvbind --devices all`). This is the
fast, toolkit-free alternative to the NVIDIA Container Toolkit for exposing the
GPU into containers — the substrate gaming containers like [[GhostForge]] rely
on.

Built on [[Rust]]; part of the [[Ghost Ecosystem]].
