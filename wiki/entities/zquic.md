---
type: entity
title: "zquic"
created: 2026-06-21
updated: 2026-06-21
tags:
  - quic
  - networking
  - zig
  - transport
status: seed
entity_type: repository
language: Zig
repo_status: active
purpose: "Modular, high-performance QUIC transport stack built entirely in Zig"
related:
  - "[[Zig]]"
  - "[[zcrypto]]"
  - "[[zsync]]"
  - "[[GhostChain]]"
  - "[[GhostKellz]]"
---

# zquic

A modular, high-performance QUIC transport stack built entirely in [[Zig]]
(0.17.0-dev line). By [[GhostKellz]].

## Highlights

- Native async runtime with no external dependencies.
- Stable QUIC / HTTP3 cryptography (via [[zcrypto]]), with explicit experimental
  post-quantum hooks.
- SSH/QUIC secret injection support.
- HTTP/3, DoQ, VPN, and service layers tuned for "Ghost workloads."

zquic is the transport backbone for networked ghost\* services (e.g.
[[GhostChain]]'s GQUIC transport), pairing with [[zcrypto]] for crypto and
[[zsync]] for async.

Built on [[Zig]].
