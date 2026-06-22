---
type: concept
title: "Ghost Ecosystem"
created: 2026-06-21
updated: 2026-06-21
tags:
  - ecosystem
  - ghost
  - umbrella
status: seed
related:
  - "[[ghostctl]]"
  - "[[Ghostlang]]"
  - "[[GhostChain]]"
  - "[[GhostWire]]"
  - "[[NV Tools Suite]]"
  - "[[Zig]]"
  - "[[Rust]]"
  - "[[GhostKellz]]"
---

# Ghost Ecosystem

The **ghost\*** family is [[GhostKellz]]'s sprawling collection of Linux/dev/
infra tools sharing a naming convention, branding (Tokyo Night aesthetics,
"👻" motifs), and a split-language strategy: [[Rust]] for application/service
tooling, [[Zig]] for low-level engines and libraries.

> [!key-insight]
> "Ghost" is a brand, not a single product. The repos span scripting languages,
> blockchains, VPNs, terminals, backup tools, package managers, and desktop apps —
> loosely connected by shared infra (e.g. [[Zig]] crypto/transport stacks) and a
> common author.

## Languages & runtimes

- [[Ghostlang]] — embedded Lua-like scripting engine in Zig (a.k.a. `.gza`).
- [[Grove]] — Tree-sitter wrapper for Zig powering syntax highlighting.
- [[ghostls]] — Ghostlang language server (Zig).
- `tree-sitter-ghostlang` — the Tree-sitter grammar for Ghostlang.
- [[GShell]] — next-gen shell scriptable in Ghostlang.
- [[Grim]] — Zig-powered Vim-soul editor/IDE (uses Grove + Ghostlang).

## System administration & infra

- [[ghostctl]] — all-in-one sysadmin/DevOps/homelab toolkit (Rust).
- [[ghostbrew]] / [[GhostView]] — Arch package discovery/management.
- [[GhostWire]] — zero-trust WireGuard mesh VPN (Rust control plane, Zig tools).
- [[Ghostwarden]] — nftables/bridge/VM network guardian.
- [[GhostCP]] — host-level hosting control panel (Axum + Leptos).
- [[Wraith]] — Zig web server / reverse proxy.

## Storage, registries & backup

- [[ghostcrate]] — self-hosted Rust crate registry.
- [[ghostsnap]] — deduplicating backup CLI.
- [[ghostport]] — Docker/OCI registry with optional SvelteKit UI.

## Web5 / blockchain

- [[GhostChain]] — Rust-microservices Web5 blockchain (GQUIC transport, PQ
  crypto, GhostPlane L2).

## Desktop, terminal & media

- [[ghostshell]] — terminal emulator (Ghostty-derived).
- [[GhostForge]] — gaming platform manager on the Bolt runtime.
- [[ghostVK]] — Zig Vulkan runtime for HDR/high-refresh gaming.
- [[GhostWave]] — RTX-voice-style noise suppression for Linux.
- [[GhostPanel]] — "Portainer for Bolt" container management UI.
- ghostpad, ghostview, ghostfetch, ghost-kde, ghostwin, ghoststream, ghostcast,
  ghostlink — smaller desktop/utility/remote-access tools.

## Cross-cutting tech

Many ghost\* services lean on the Zig infra stack: [[zquic]] (QUIC transport),
[[zcrypto]] (crypto, incl. post-quantum), [[zsync]] (async runtime), and
[[zqlite]] (embedded SQL). See [[NV Tools Suite]] for the GPU-focused sibling
family and [[Zig]] / [[Rust]] for the language foundations.
