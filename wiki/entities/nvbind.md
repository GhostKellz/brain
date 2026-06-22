---
type: entity
title: "nvbind"
created: 2026-06-21
updated: 2026-06-21
tags:
  - nvidia
  - gpu
  - containers
  - rust
status: seed
entity_type: repository
language: Rust
repo_status: active
purpose: "Lightning-fast Rust alternative to the NVIDIA Container Toolkit for GPU passthrough"
related:
  - "[[Rust]]"
  - "[[nvcontrol]]"
  - "[[NV Tools Suite]]"
  - "[[GhostKellz]]"
---

# nvbind

A fast, [[Rust]]-based alternative to the NVIDIA Container Toolkit — next-gen GPU
passthrough for modern container workflows. Part of the [[NV Tools Suite]]; by
[[GhostKellz]].

Where [[nvcontrol]] manages the GPU on the host, nvbind focuses on exposing the
GPU into containers, making it the container-runtime counterpart in the nv\*
family (and relevant to Bolt-based gaming containers like [[GhostForge]]).

Built on [[Rust]].
