---
type: entity
title: "ghostVK"
created: 2026-06-21
updated: 2026-06-21
tags:
  - vulkan
  - zig
  - gaming
  - gpu
status: seed
entity_type: repository
language: Zig
repo_status: experimental
purpose: "Next-generation Vulkan runtime in Zig for high-refresh-rate HDR gaming on Linux with NVIDIA GPUs"
related:
  - "[[Zig]]"
  - "[[nvvk]]"
  - "[[NV Tools Suite]]"
  - "[[Ghost Ecosystem]]"
  - "[[GhostKellz]]"
---

# ghostVK

A next-generation Vulkan runtime written in [[Zig]], purpose-built for
high-refresh-rate HDR gaming on Linux with NVIDIA GPUs. Marked experimental. By
[[GhostKellz]].

## Goals

- Sub-1ms frame times at 1440p/360Hz.
- HDR-first design: native PQ/ST2084 EOTF and HDR metadata.
- Targets the modern Linux graphics stack: Wayland, DRM/KMS, NVIDIA Open Kernel
  Modules.

Bridges the [[Ghost Ecosystem]] and the [[NV Tools Suite]]; related to the
lower-level extension library [[nvvk]].

Built on [[Zig]].
