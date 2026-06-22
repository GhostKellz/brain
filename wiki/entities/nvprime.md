---
type: entity
title: "nvprime"
created: 2026-06-21
updated: 2026-06-21
tags:
  - zig
  - nvidia
  - gpu
  - wayland
  - vulkan
status: seed
entity_type: repository
language: Zig
repo_status: experimental
purpose: "Unified NVIDIA subsystem layer for Linux — gaming, workstation, AI/compute, streaming, and creative workflows"
related:
  - "[[Zig]]"
  - "[[NV Tools Suite]]"
  - "[[nvvk]]"
  - "[[ghostVK]]"
  - "[[GhostKellz]]"
---

# nvprime

NVPrime is a comprehensive **NVIDIA subsystem layer for Linux** written in
[[Zig]] — not just gaming, not just drivers, but a unified platform across
gaming, workstation, AI/compute, streaming, and creative workflows. Conceptually
"AMD's ROCm, but broader." Marked experimental. By [[GhostKellz]].

## Vision

A layered platform sitting beneath games, AI, creative apps, and streaming, with
a Ghostflare UI/HUD on top and the NVPrime platform underneath driving the NVIDIA
stack. Targets NVIDIA driver 580+ on Linux x86_64.

## Where it fits

The umbrella/foundation of the [[NV Tools Suite]] — broader in scope than the
individual nv* tools, tying together the Vulkan layer ([[nvvk]], [[ghostVK]]) and
the rest of the NVIDIA-on-Linux story. Built on [[Zig]].
