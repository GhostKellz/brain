---
type: entity
title: GhostWave
created: 2026-06-21
updated: 2026-06-28
tags:
  - audio
  - nvidia
  - rtx
  - linux
  - rust
status: developing
entity_type: repository
language: Rust
repo_status: experimental
purpose: NVIDIA RTX-powered AI noise-suppression and audio-processing engine for
  Linux with PipeWire integration
related:
  - "[[PhantomLink]]"
  - "[[Nitrogen]]"
  - "[[Rust]]"
  - "[[NVIDIA]]"
  - "[[NV Tools Suite]]"
  - "[[Ghost Ecosystem]]"
  - "[[GhostKellz]]"
---

# GhostWave

An [[NVIDIA]] RTX-powered AI noise-suppression and audio-processing engine for
Linux with PipeWire integration — an NVIDIA-Broadcast-equivalent. Authored by
[[GhostKellz]], MIT-licensed, written in [[Rust]] (2024 edition, 1.85+,
v0.3.x).

## Language & stack

- **Language**: Rust 2024.
- **Workspace**: `ghostwave-core` and `ghostwave-vst`.
- **Audio backends**: PipeWire, ALSA, JACK, and CPAL.
- **Acceleration**: optional NVIDIA CUDA.
- **Plugin host**: VST3 / CLAP via nih-plug with egui.
- **Runtime/DSP**: Tokio; `dasp` for DSP.

## Capabilities

- **DSP pipeline**: high-pass → voice activity detection → spectral denoise →
  noise gate/expander → soft limiter with lookahead.
- **Profiles**: Balanced, Streaming, and Studio.
- **Virtual source**: creates a "GhostWave Clean" PipeWire source.
- **Hardware auto-detect**: XLR interfaces and NVIDIA GPU capabilities.
- **RTX detection**: 50-series (FP4), 40/30/20-series (FP16), with CUDA kernels;
  TensorRT/ONNX support planned.
- **Control**: JSON-RPC over UNIX sockets; CLI (`--doctor`, `--bench`); systemd
  user service.
- **Plugin**: VST3 / CLAP build for use inside DAWs.
- **Bridge**: integration with [[PhantomLink]].

> [!note]
> The GPU inference framework is present, but CPU spectral denoise is the current
> production path; neural inference is in development.

## Relationship

GhostWave is the underlying engine consumed by [[PhantomLink]] and [[Nitrogen]].
Part of the [[NV Tools Suite]] and the wider [[Ghost Ecosystem]].

## Status

v0.3.x, open beta. MIT-licensed.
