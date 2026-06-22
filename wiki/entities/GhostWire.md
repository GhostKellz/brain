---
type: entity
title: "GhostWire"
created: 2026-06-21
updated: 2026-06-21
tags:
  - vpn
  - wireguard
  - networking
  - rust
  - zig
status: seed
entity_type: repository
language: Rust
repo_status: active
purpose: "Next-generation WireGuard mesh VPN — Rust control plane, Zig tools, zero-trust by default"
related:
  - "[[Rust]]"
  - "[[Zig]]"
  - "[[Ghostwarden]]"
  - "[[Ghost Ecosystem]]"
  - "[[GhostKellz]]"
---

# GhostWire

A self-hosted, zero-trust overlay network inspired by Tailscale/Headscale,
rebuilt from scratch in [[Rust]] with lightweight tooling in [[Zig]]. By
[[GhostKellz]]; part of the [[Ghost Ecosystem]].

## Architecture

- **Data plane**: WireGuard.
- **NAT traversal**: QUIC/DERP-style relays.
- **Control plane**: clean Rust control plane with OIDC authentication.
- **Tooling**: lightweight Zig utilities.

Pairs with [[Ghostwarden]] on the network-policy/firewall side. A clear example
of the ecosystem's Rust-for-services / Zig-for-tools split.
