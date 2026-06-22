---
type: entity
title: "zcrypto"
created: 2026-06-21
updated: 2026-06-21
tags:
  - cryptography
  - zig
  - library
  - post-quantum
status: seed
entity_type: repository
language: Zig
repo_status: active
purpose: "Modular cryptography library for Zig — core primitives, QUIC helpers, and feature-gated transport integrations"
related:
  - "[[Zig]]"
  - "[[zquic]]"
  - "[[zqlite]]"
  - "[[GhostChain]]"
  - "[[GhostKellz]]"
---

# zcrypto

A modular cryptography library written in [[Zig]]. The stable release centers on
core primitives, QUIC helpers, and feature-gated transport/runtime integrations
verified in-repo. By [[GhostKellz]].

## Role

zcrypto is the shared crypto foundation for the Zig half of the ecosystem:

- Provides QUIC crypto helpers consumed by [[zquic]].
- Supplies the post-quantum primitives that [[zqlite]] references in its
  experimental crypto scaffolding.
- Underpins crypto in [[GhostChain]]'s transport.

Built on [[Zig]].
