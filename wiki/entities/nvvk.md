---
type: entity
title: "nvvk"
created: 2026-06-21
updated: 2026-06-21
tags:
  - nvidia
  - vulkan
  - zig
  - library
  - gaming
status: seed
entity_type: repository
language: Zig
repo_status: experimental
purpose: "Zig library of NVIDIA Vulkan extension wrappers with C ABI for DXVK/vkd3d-proton integration"
related:
  - "[[Zig]]"
  - "[[nvlatency]]"
  - "[[ghostVK]]"
  - "[[NV Tools Suite]]"
  - "[[GhostKellz]]"
---

# nvvk

A [[Zig]] library providing optimized NVIDIA Vulkan extension wrappers with C ABI
exports, for integration with DXVK, vkd3d-proton, and other Vulkan translation
layers. Marked experimental. Part of the [[NV Tools Suite]]; by [[GhostKellz]].

Exposes underused NVIDIA-specific extensions such as `VK_NV_low_latency2`
(Reflex, used by [[nvlatency]]) and `VK_NV_device_diagnostics_config` (crash
diagnostics). The C ABI surface is the bridge that lets translation layers call
into it.

Built on [[Zig]]; related to the larger Vulkan runtime [[ghostVK]].
