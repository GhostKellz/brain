---
type: entity
title: "Proton-NV"
created: 2026-06-21
updated: 2026-06-21
tags:
  - nvidia
  - proton
  - gaming
status: seed
entity_type: repository
language: Shell
repo_status: experimental
purpose: "Custom Proton build tuned for NVIDIA GPUs on Linux (open kernel modules 595+)"
related:
  - "[[nvproton]]"
  - "[[NV Tools Suite]]"
  - "[[GhostKellz]]"
---

# Proton-NV

A custom Proton build specifically tuned for NVIDIA GPUs on Linux. Marked
experimental. Part of the [[NV Tools Suite]]; by [[GhostKellz]].

Targets:

- NVIDIA Open Kernel Module 595+ with GSP=1 (595.45.04+ recommended)
- vkd3d-proton 3.0 with NVIDIA-specific optimizations + `VK_EXT_descriptor_heap`
- RTX 40/50 series GPUs
- CachyOS / Linux-tkg kernels with BORE+EEVDF scheduler

Complements [[nvproton]] (the integration layer) by being the optimized Proton
runtime itself.
