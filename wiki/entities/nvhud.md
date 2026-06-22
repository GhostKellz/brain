---
type: entity
title: "nvhud"
created: 2026-06-21
updated: 2026-06-21
tags:
  - nvidia
  - gpu
  - osd
  - zig
  - gaming
status: seed
entity_type: repository
language: Zig
repo_status: active
purpose: "GPU-accelerated performance OSD for NVIDIA Linux gaming — a MangoHud alternative in Zig"
related:
  - "[[Zig]]"
  - "[[nvcontrol]]"
  - "[[NV Tools Suite]]"
  - "[[GhostKellz]]"
---

# nvhud

A next-generation performance overlay (OSD) for NVIDIA Linux gaming, built in
[[Zig]] as a MangoHud alternative with NVIDIA-specific optimizations, lower
overhead, and deeper integration with the [[NV Tools Suite]]. By [[GhostKellz]].

Optimized for recent NVIDIA driver branches (590+), with smooth overlay
injection during Vulkan swapchain operations. The OSD complement to the
host-side [[nvcontrol]].

Built on [[Zig]].
