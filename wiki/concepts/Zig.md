---
type: concept
title: "Zig"
created: 2026-06-21
updated: 2026-06-21
tags:
  - language
  - systems-programming
  - zig
status: seed
related:
  - "[[zqlite]]"
  - "[[Ghostlang]]"
  - "[[Grove]]"
  - "[[Ghost Ecosystem]]"
  - "[[NV Tools Suite]]"
---

# Zig

Zig is a general-purpose systems programming language and toolchain aimed at
robustness, optimality, and maintainability. It positions itself as a modern
alternative to C: no hidden control flow, no hidden memory allocations, manual
memory management with explicit allocators, comptime metaprogramming, and a
C/C++ cross-compiler bundled into the toolchain.

## Why it matters here

Zig is the primary "low-level" language across [[GhostKellz]]'s ecosystem. Where
[[Rust]] is used for application-level tooling and services, Zig is reached for
when the project wants zero external runtime dependencies, a small footprint,
explicit allocator control, or a C ABI surface for embedding.

## Key characteristics relevant to these projects

- **Explicit allocators** — every allocation passes an allocator, which makes
  memory-leak testing first-class (see [[zqlite]]'s leak-detection test suites).
- **comptime** — compile-time code execution used for generic data structures
  and configuration without macros.
- **C interop / C ABI** — Zig can both consume C headers and export a C ABI,
  enabling FFI bindings (e.g. [[zqlite]] C API, [[nvvk]] C ABI exports).
- **Single toolchain** — `zig build` drives compilation, testing, and
  cross-compilation; build logic is written in Zig itself (`build.zig`).
- **No package runtime deps** — `build.zig.zon` declares dependencies fetched by
  `zig fetch`; several projects deliberately ship with zero external packages.

> [!key-insight]
> The ecosystem tracks Zig **nightly/dev** releases (e.g. `0.17.0-dev`), not just
> stable tags. Project `build.zig.zon` files pin a `minimum_zig_version` to a
> specific dev build, so the toolchain version is part of each repo's contract.

## Projects in this vault built in Zig

- **Databases / storage**: [[zqlite]]
- **Languages / runtimes**: [[Ghostlang]], [[Grove]], [[GShell]], [[ghostls]]
- **GPU / graphics**: [[nvhud]], [[nvvk]], [[ghostVK]]
- **Infra / transport**: [[zquic]], [[zcrypto]], [[zsync]], [[Wraith]]
- **TUI / tooling**: [[Phantom]]

See [[Ghost Ecosystem]] and [[NV Tools Suite]] for the broader family maps. The
Rust-side counterparts are tracked under [[Rust]].
