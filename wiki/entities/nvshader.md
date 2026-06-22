---
type: entity
title: "nvshader"
created: 2026-06-21
updated: 2026-06-21
tags:
  - nvidia
  - gpu
  - shader-cache
  - gaming
status: seed
entity_type: repository
language: Rust
repo_status: active
purpose: "NVIDIA shader cache management and pre-warming to eliminate shader-compilation stutter on Linux"
related:
  - "[[nvproton]]"
  - "[[NV Tools Suite]]"
  - "[[GhostKellz]]"
---

# nvshader

A shader cache management and optimization system for Linux gaming. It tackles
the "shader compilation stutter" problem with a unified interface across DXVK,
vkd3d-proton, Mesa, and the NVIDIA driver caches, plus Fossilize pre-compilation
and real-time monitoring (inotify). Part of the [[NV Tools Suite]]; by
[[GhostKellz]].

Wired into Proton workflows by [[nvproton]].
