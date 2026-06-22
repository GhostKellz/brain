---
type: entity
title: "GhostForge"
created: 2026-06-21
updated: 2026-06-21
tags:
  - gaming
  - linux
  - containers
  - rust
status: seed
entity_type: repository
language: Rust
repo_status: experimental
purpose: "Modern Lutris-alternative gaming platform manager built on the Bolt container runtime"
related:
  - "[[Bolt]]"
  - "[[Ghost Ecosystem]]"
  - "[[GhostKellz]]"
---

# GhostForge

A next-generation gaming platform manager for Linux, designed to replace Lutris
using modern container technology. Built on the [[Bolt]] gaming container
runtime, it provides isolated gaming environments, GPU passthrough, real-time
monitoring, and Wine/Proton integration. By [[GhostKellz]]; part of the
[[Ghost Ecosystem]].

The container runtime is [[Bolt]] (also the platform behind [[GhostPanel]]),
and GPU passthrough comes from Bolt's built-in NVIDIA path (formerly the
standalone `nvbind`).
