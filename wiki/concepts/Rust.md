---
type: concept
title: "Rust"
created: 2026-06-21
updated: 2026-06-28
tags:
  - language
  - systems-programming
  - rust
status: developing
related:
  - "[[Zig]]"
  - "[[Go]]"
  - "[[Python]]"
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

## Where it fits vs Go / Zig

| | Rust | [[Go]] | [[Zig]] |
|--|------|--------|---------|
| Memory | ownership/borrow, no GC | GC | manual + allocators |
| Safety | compile-time guaranteed | runtime + GC | manual (defer/errors) |
| Concurrency | async/await + `Send`/`Sync` | goroutines + channels | manual |
| Compile speed | slow | very fast | fast |
| Best at | safety-critical apps, daemons, services | CLIs, cloud-native tooling | low-level engines, C ABI |

## Strengths / trade-offs

**Strengths**
- Memory + thread safety at compile time, zero runtime cost (see
  [[Rust Ownership and Borrowing]]) — no GC pauses, no data races.
- C-class performance with high-level ergonomics (iterators, pattern matching,
  traits, `Result`/`Option`, exhaustive `match`).
- First-class tooling: `cargo`, `clippy`, `rustfmt`, `rust-analyzer` (see
  [[Cargo Workflow]]); rich crate ecosystem.
- Fearless refactoring — the type system + borrow checker catch breakage.

**Trade-offs**
- Steep learning curve; the borrow checker rejects unsafe-but-familiar patterns
  until the model clicks.
- Slow compiles relative to [[Go]]/[[Zig]].
- More ceremony than GC languages for quick prototyping (reach for [[Python]]).
- `async` Rust has real complexity (lifetimes, `Pin`, runtime choice).

## Tooling

The toolchain — `rustup`, `rustc`, and `cargo` (build + deps + test + lint +
publish) — is covered in **[[Cargo Workflow]]**.

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
