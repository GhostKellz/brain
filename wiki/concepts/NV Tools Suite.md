---
type: concept
title: "NV Tools Suite"
created: 2026-06-21
updated: 2026-06-21
tags:
  - nvidia
  - gpu
  - linux
  - gaming
  - umbrella
status: seed
related:
  - "[[nvcontrol]]"
  - "[[Bolt]]"
  - "[[nvhud]]"
  - "[[nvsync]]"
  - "[[nvshader]]"
  - "[[Ghost Ecosystem]]"
  - "[[GhostKellz]]"
---

# NV Tools Suite

The **nv\*** family is [[GhostKellz]]'s set of NVIDIA-on-Linux tools, each
targeting a specific pain point of Linux gaming and GPU management. They share a
common theme: bring Windows-class NVIDIA features (Reflex, vibrance, VRR, shader
caching) to Linux + Wayland, tuned for the NVIDIA open kernel modules.

> [!key-insight]
> The suite is modular — one narrow tool per niche — with [[nvproton]] designed
> as the "glue" that wires them into Steam Proton/Wine, and [[nvprime]] framed as
> a unified platform layer. Languages split between [[Rust]] (app tooling) and
> [[Zig]] (low-level libs).

## The tools

- **[[nvcontrol]]** — flagship NVIDIA settings manager (vibrance, VRR, HDR,
  overclock, fan, power) for Wayland. Rust, TUI + GUI.
- **GPU passthrough** — the fast, toolkit-free alternative to the NVIDIA
  Container Toolkit is now **built into [[Bolt]]** (the former standalone
  `nvbind` tool, folded in as a native runtime path).
- **[[nvhud]]** — GPU-accelerated performance OSD (MangoHud alternative) in Zig.
- **[[nvsync]]** — VRR & G-Sync manager for Linux (Wayland + X11).
- **[[nvshader]]** — shader cache management/pre-warming across DXVK,
  vkd3d-proton, Mesa, and the NVIDIA driver to kill shader-compile stutter.
- **[[nvlatency]]** — NVIDIA Reflex / frame-latency measurement and reduction.
- **[[nvvk]]** — Zig library of NVIDIA Vulkan extension wrappers with C ABI
  (e.g. `VK_NV_low_latency2`) for DXVK/vkd3d-proton.
- **[[nvproton]]** — integration layer wiring all nv\* tools into Proton/Wine.
- **[[Proton-NV]]** — custom Proton build tuned for NVIDIA open modules 595+.
- **nvprime** — "Unified NVIDIA Linux Platform" subsystem layer.
- **nvfury** — additional nv\* utility (early/branding-stage).

## Common context

- Track recent NVIDIA driver branches (590+/595+/610 open kernel modules).
- Wayland-first, with X11 fallback where relevant.
- Frequently overlap with the gaming side of the [[Ghost Ecosystem]]
  ([[GhostForge]], [[ghostVK]], [[GhostWave]]).

See [[nvcontrol]] for the anchor project and [[Zig]] / [[Rust]] for the language
foundations.
