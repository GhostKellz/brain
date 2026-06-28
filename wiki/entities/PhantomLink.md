---
type: entity
title: PhantomLink
created: 2026-06-21
updated: 2026-06-28
tags:
  - audio
  - linux
  - rust
  - nvidia
status: developing
entity_type: repository
language: Rust
repo_status: experimental
purpose: Professional audio mixer and interface control for Linux with RTX AI
  noise suppression
related:
  - "[[GhostWave]]"
  - "[[Rust]]"
  - "[[NVIDIA]]"
  - "[[NV Tools Suite]]"
  - "[[Ghost Ecosystem]]"
  - "[[GhostKellz]]"
---

# PhantomLink

A professional audio mixer and interface control application for Linux with RTX
AI noise suppression — an Elgato Wavelink-XLR-equivalent for Linux. Authored by
[[GhostKellz]], MIT-licensed, written in [[Rust]] (2024 edition, v0.4.x).

## Language & stack

- **Language**: Rust 2024.
- **GUI**: egui / eframe.
- **Audio**: CPAL with ALSA, PipeWire, and JACK backends.
- **Acceleration**: optional [[NVIDIA]] CUDA.

## Capabilities

- **Interface control**: full Focusrite Scarlett Solo 4th Gen control — 48V
  phantom power, Air mode, line/instrument switching, direct monitoring, and
  12-channel hardware metering.
- **Mixer UI**: gain knobs, VU meters with peak hold, and clip detection.
- **Effects**: per-channel VST hosting, plus mute/solo/pan.
- **AI denoising**: routed through [[GhostWave]] (RTX FP4/FP16 with CPU
  fallback), with voice activity detection, echo cancellation, and
  Fast/Balanced/Quality/Ultra profiles.
- **Theming**: multiple UI themes.
- **Packaging**: AUR, COPR, `.deb`, and AppImage.
- **Display servers**: [[Wayland]] and X11.

## Relationship

PhantomLink consumes [[GhostWave]] as its denoising engine. Part of the
[[Ghost Ecosystem]] desktop/media tooling, alongside the [[NV Tools Suite]].

## Status

v0.4.x, experimental. MIT-licensed.
