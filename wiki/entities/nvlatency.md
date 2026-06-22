---
type: entity
title: "nvlatency"
created: 2026-06-21
updated: 2026-06-21
tags:
  - nvidia
  - gpu
  - reflex
  - latency
  - gaming
status: seed
entity_type: repository
language: Rust
repo_status: experimental
purpose: "NVIDIA Reflex and frame-latency measurement/reduction toolkit for Linux"
related:
  - "[[nvvk]]"
  - "[[nvproton]]"
  - "[[NV Tools Suite]]"
  - "[[GhostKellz]]"
---

# nvlatency

A toolkit for measuring, analyzing, and reducing input-to-display latency on
NVIDIA GPUs under Linux — bringing Windows-level Reflex functionality to Linux
gaming. Marked experimental. Part of the [[NV Tools Suite]]; by [[GhostKellz]].

Optimized for NVIDIA 590+ drivers and `VK_NV_low_latency2`; pairs with [[nvvk]]
(the Vulkan extension library) and is integrated into Proton via [[nvproton]].
