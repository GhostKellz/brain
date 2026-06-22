---
type: concept
title: "Rust"
created: 2026-06-21
updated: 2026-06-21
tags:
  - language
  - systems-programming
  - rust
status: seed
related:
  - "[[Zig]]"
  - "[[Rust Ownership and Borrowing]]"
  - "[[Cargo Workflow]]"
  - "[[Ghost Ecosystem]]"
  - "[[NV Tools Suite]]"
---

# Rust

Rust is a systems programming language focused on safety, concurrency, and
performance. Its ownership-and-borrowing model (see [[Rust Ownership and
Borrowing]]) enforces memory safety at compile time without a garbage collector,
making it the application-and-services half of this ecosystem's two-language
strategy.

## Why it matters here

Rust is the primary "application" language across [[GhostKellz]]'s ecosystem.
Where [[Zig]] is reached for engines, runtimes, and zero-dependency low-level
work, Rust is used for user-facing tools, daemons, control planes, and network
services where a richer crate ecosystem and strong type guarantees pay off.

## The two-language thesis

> [!key-insight]
> The unifying technical thesis is *local-first infrastructure, **Rust for apps**,
> [[Zig]] for engines*. The split is deliberate: Rust where ecosystem and safety
> ergonomics win, Zig where explicit allocation and a C ABI matter.

## Projects in this vault built in Rust

- **AI / agents**: [[Jarvis]], [[Zeke]]
- **GPU / NVIDIA**: [[nvcontrol]]
- **Infra / containers**: [[Bolt]], [[ghostctl]]
- **Networking / chain**: [[GhostWire]], [[GhostChain]]
- **Hosting / registry**: [[GhostCP]], [[ghostport]]
- **Crates / tooling**: [[ghostcrate]], [[reaper]]

Workflow conventions live in [[Cargo Workflow]]. The Zig-side counterparts are
tracked under [[Zig]]. See [[Ghost Ecosystem]] and [[NV Tools Suite]] for the
broader family maps.
