---
type: entity
title: nvcontrol
created: 2026-06-21
updated: 2026-06-28
tags:
  - nvidia
  - gpu
  - linux
  - wayland
  - rust
status: developing
entity_type: repository
language: Rust
repo_status: active
purpose: Comprehensive NVIDIA GPU control tool for Linux with native Wayland
  support — vibrance, VRR/G-SYNC, HDR, monitoring, overclocking, fan and power
  control
related:
  - "[[Rust]]"
  - "[[NV Tools Suite]]"
  - "[[NVIDIA]]"
  - "[[NVIDIA GSP Firmware]]"
  - "[[Wayland]]"
  - "[[Local LLM Inference]]"
  - "[[GhostKellz]]"
---

# nvcontrol

A comprehensive NVIDIA GPU control tool for Linux, built from the ground up for
[[Wayland]]. It covers display and color, monitoring, overclocking, fan and power
control, driver diagnostics, and CUDA/AI tooling. Authored by [[GhostKellz]] /
CK Technology LLC, MIT-licensed, written in [[Rust]].

## Language & stack

- **Language**: Rust 1.95+ (2024 edition).
- **TUI**: ratatui dashboard.
- **GUI**: egui application.
- **CLI**: `nvctl`.
- GPU access via `nvml-wrapper`; pure-Rust digital vibrance through NVKMS ioctls
  (no `nvidia-settings` dependency).

## Architecture & approach

- **Native NVKMS ioctls** for digital vibrance, so color control works under
  [[Wayland]] without `nvidia-settings`.
- Driver diagnostics align kernel, userspace, and [[NVIDIA GSP Firmware]]
  versions, with redacted support bundles for shareable artifacts.

## Capabilities

- **Display & color**: digital vibrance (0–200), full/limited RGB, color space
  (RGB/YCbCr), dithering, sharpening; VRR / G-SYNC toggle; HDR on [[KDE Plasma]]
  6, GNOME 45+, Hyprland, and COSMIC.
- **Monitoring**: `nvctl gpu info/stat/nvtop`, live watch, and a TUI with
  Overview, Performance, Memory, Temp, Power, Processes, Overclock, Fan,
  Profiles, Tuner, Profiler, OSD, Drivers, DLSS, CUDA-AI, and Settings tabs.
- **Overclock & power** (experimental): clock offsets, per-fan and auto fan
  control, power limits, persistent mode, and power-curve profiles.
- **Driver diagnostics**: `driver diagnose-release` (kernel/userspace/GSP
  alignment, JSON output), `support-bundle` (redacted), and `doctor --support`;
  a compatibility matrix for NVIDIA 610+ open-kernel modules across RTX 50/40/30/
  20 and GTX legacy.
- **CUDA / AI**: `cuda doctor/ollama/env/smoke` and VRAM-fit analysis (see
  [[Local LLM Inference]]).
- **ASUS ROG Power Detector+**: 12V-2x6 connector health monitoring.
- **Gaming**: detection, launcher and profiles, Gamescope integration,
  passthrough diagnostics, latency tooling, and DLSS detection.

## Hardware & platform support

- RTX 50, 40, 30, and 20 series supported; GTX legacy parts covered for basic
  control.
- Targets NVIDIA 610+ open kernel modules.

> [!key-insight]
> Color, vibrance, and VRR are driven natively through NVKMS ioctls rather than
> by shelling out to `nvidia-settings`, which is what makes them work under
> Wayland.

## Status

v0.8.x, well-maintained and actively developed. Overclock and power features are
explicitly experimental. MIT-licensed.

## Relationships

Part of the [[NV Tools Suite]]. Built on [[Rust]]; integrates with the wider
[[NVIDIA]] stack on Linux.
