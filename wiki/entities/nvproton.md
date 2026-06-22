---
type: entity
title: "nvproton"
created: 2026-06-21
updated: 2026-06-21
tags:
  - nvidia
  - proton
  - gaming
  - integration
status: seed
entity_type: repository
language: Rust
repo_status: experimental
purpose: "Integration layer that bridges all nv* tools with Steam Proton/Wine for per-game optimization"
related:
  - "[[nvcontrol]]"
  - "[[nvshader]]"
  - "[[nvlatency]]"
  - "[[Proton-NV]]"
  - "[[NV Tools Suite]]"
  - "[[GhostKellz]]"
---

# nvproton

The "glue" of the [[NV Tools Suite]]: an integration layer connecting all nv\*
tools with Steam Proton/Wine for automatic game optimization, Reflex injection,
shader management, and per-game configurations. Marked experimental. By
[[GhostKellz]].

It ties together [[nvshader]] (shader caches), [[nvlatency]]/[[nvvk]] (Reflex/
latency), and host settings from [[nvcontrol]]. Distinct from [[Proton-NV]],
which is a custom Proton build rather than an integration layer.
