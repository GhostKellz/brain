---
type: entity
title: Nitrogen
created: 2026-06-28
updated: 2026-06-28
tags:
  - nvenc
  - streaming
  - wayland
  - nvidia
  - rust
  - screen-capture
status: developing
entity_type: repository
language: Rust
repo_status: active
purpose: Wayland-native NVIDIA NVENC screen-capture and streaming app with
  hardware encoding and frame interpolation
related:
  - "[[Rust]]"
  - "[[NVIDIA]]"
  - "[[Wayland]]"
  - "[[nvcontrol]]"
  - "[[GhostWave]]"
  - "[[NV Tools Suite]]"
  - "[[GhostKellz]]"
  - "[[CK Technology LLC]]"
---

# Nitrogen

A Wayland-native NVIDIA NVENC screen-capture and streaming application — to
Discord, Twitch, YouTube, or local recording — with hardware encoding and frame
interpolation. Authored by [[GhostKellz]] / [[CK Technology LLC]],
MIT-licensed, written in [[Rust]].

> [!key-insight]
> Capture goes through `xdg-desktop-portal` and PipeWire, so it is Wayland-safe
> with no XWayland dependency, while encoding is offloaded to NVENC.

## Stack

- **Language**: Rust (2024 edition).
- **Workspace**: `nitrogen-core` plus `nitrogen-cli`.
- **Runtime**: Tokio.
- **Capture**: `xdg-desktop-portal` (Wayland-safe) with PipeWire audio.
- **Encode**: FFmpeg (`libav*`) with NVENC.
- **System**: D-Bus via zbus/ashpd; `webrtc` crate; evdev hotkeys; an axum
  signaling server.

## Capabilities

- Wayland-native capture on KDE Plasma 6, Hyprland, Sway, and wlroots — no
  XWayland.
- NVENC encoding in H.264, HEVC, and AV1.
- Presets from 720p to 4K, up to 120 fps.
- Frame interpolation via NVIDIA Optical Flow (2x-4x).
- HDR-to-SDR tonemapping (Reinhard, ACES, Hable).
- PipeWire desktop and mic audio with ducking; AAC/Opus output.
- Global hotkeys and a perf/latency overlay.
- Outputs: PipeWire virtual camera for Discord, RTMP/RTMPS, SRT, and MP4/MKV
  recording.
- Gamescope / Steam Deck detection.

## Status

v0.2.0, active development. APIs may change; not production-ready yet. MIT.

## Relationships

Built on [[Rust]]. Part of the Wayland-on-NVIDIA story — see
[[NVIDIA#NVIDIA on Wayland|NVIDIA on Wayland]] and [[Wayland]] — and integrates with [[GhostWave]] for
audio and the broader [[NV Tools Suite]] (alongside [[nvcontrol]]).
