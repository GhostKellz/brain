---
type: entity
title: "nvcontrol"
created: 2026-06-21
updated: 2026-06-21
tags:
  - nvidia
  - gpu
  - linux
  - wayland
  - rust
status: seed
entity_type: repository
language: Rust
repo_status: active
purpose: "Modern NVIDIA settings manager for Linux + Wayland — vibrance, VRR/G-SYNC, HDR, overclocking, fan and power control"
related:
  - "[[Rust]]"
  - "[[NV Tools Suite]]"
  - "[[Bolt]]"
  - "[[nvhud]]"
  - "[[nvsync]]"
  - "[[GhostKellz]]"
---

# nvcontrol

The flagship of the **nv\*** family: a comprehensive NVIDIA GPU control tool for
Linux, designed from the ground up for Wayland. Think "the missing NVIDIA
Control Panel for Linux." Authored by [[GhostKellz]] / CK Technology LLC,
MIT-licensed, written in [[Rust]].

## Language & stack

- **Language**: Rust (1.95+), with a `build.rs` and PKGBUILD/packaging tree.
- **TUI**: ratatui dashboard (`nvctl gpu stat`, `nvctl nvtop`).
- **GUI**: egui application (`nvcontrol` binary, `--features gui`).
- **Binaries**: `nvctl` (CLI/TUI) and `nvcontrol` (GUI).

## Architecture & approach

- **Native NVKMS ioctls** for digital vibrance — no `nvidia-settings` required,
  so it works under Wayland. (Inspired by `nvibrant`.)
- Compositor-aware integrations: KDE (kscreen-doctor), GNOME (gsettings),
  Hyprland (hyprctl), Sway (swaymsg), Pop!_OS COSMIC (cosmic-randr).
- Targets the NVIDIA **610 open driver** branch as the primary path; ships a
  driver diagnostics + support-bundle workflow (`nvctl driver diagnose-release`,
  `nvctl doctor --support`) with redaction options for shareable artifacts.

## Notable features

- Digital vibrance via NVKMS ioctls (0-200 scale)
- Display controls: color range (full/limited RGB), color space, dithering,
  sharpening
- VRR / G-SYNC enable/disable per display
- GPU monitoring: info, live watch, htop-style `nvtop`, full TUI dashboard
- Overclocking (experimental), fan control, power management (experimental)
- ASUS ROG support: **Power Detector+** for 12V-2x6 connector health on ROG
  Astral/Matrix RTX 5090
- Shell completions (bash/zsh/fish); multiple themes (Tokyo Night default)

## Hardware & platform support

- RTX 50 (Blackwell), 40 (Ada), 30 (Ampere) fully supported; 20 (Turing)
  supported; GTX 16/10 basic.
- Requires NVIDIA driver 610+ (open kernel modules), kernel 6.6+ (7.0+
  recommended).
- Arch Linux is the premier platform (AUR `nvcontrol-git`); Fedora/Nobara/
  Bazzite/Pop!_OS Tier 1; Debian/Ubuntu full.

> [!key-insight]
> The differentiator is doing color/vibrance/VRR natively through NVKMS ioctls
> rather than shelling out to `nvidia-settings`, which is what makes it work on
> Wayland where the old control panel does not.

## Status

Active. Overclocking, voltage, and power-limit features are explicitly flagged
experimental ("use at your own risk").

## Relationships

The anchor of the [[NV Tools Suite]]. Sibling tools cover adjacent niches:
[[nvhud]] (performance OSD), [[nvsync]] (VRR/G-Sync manager), and others.
Container GPU passthrough now lives natively in [[Bolt]] (formerly the
standalone `nvbind`). Built on [[Rust]].
