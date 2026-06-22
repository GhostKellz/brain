---
type: entity
title: "nvsync"
created: 2026-06-21
updated: 2026-06-21
tags:
  - nvidia
  - gpu
  - vrr
  - gaming
status: seed
entity_type: repository
language: Rust
repo_status: experimental
purpose: "NVIDIA VRR & G-Sync manager for Linux with Wayland and X11 support"
related:
  - "[[nvcontrol]]"
  - "[[NV Tools Suite]]"
  - "[[GhostKellz]]"
---

# nvsync

An NVIDIA Variable Refresh Rate manager for Linux, providing G-Sync, G-Sync
Compatible, and VRR support with full Wayland and X11 support. Marked
experimental. Part of the [[NV Tools Suite]]; by [[GhostKellz]].

Optimized for NVIDIA 595+ open kernel modules and modern Wayland compositors.
Overlaps with the VRR/G-Sync features in [[nvcontrol]], offering a dedicated tool
for that niche.
