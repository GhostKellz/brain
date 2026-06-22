---
type: entity
title: "Ghostlang"
created: 2026-06-21
updated: 2026-06-21
tags:
  - language
  - scripting
  - zig
  - embedded
status: seed
entity_type: repository
language: Zig
repo_status: active
purpose: "Lightweight embedded scripting engine in Zig — a Lua replacement with JS-compatible syntax, sandboxing, FFI and optional JIT"
related:
  - "[[Zig]]"
  - "[[Grove]]"
  - "[[ghostls]]"
  - "[[GShell]]"
  - "[[Grim]]"
  - "[[Ghost Ecosystem]]"
  - "[[GhostKellz]]"
---

# Ghostlang

A lightweight embedded scripting engine written in [[Zig]], pitched as a modern
replacement for Lua. It offers Lua-like syntax with JavaScript compatibility, a
register-based VM, sandboxing, and a bidirectional FFI. Files use the `.gza`
extension. Part of the [[Ghost Ecosystem]]; by [[GhostKellz]].

## Features

- **Embedded scripting** for config, automation, plugins, and game scripting.
- **Lightweight register-based VM** with a small constant footprint.
- **FFI** — bidirectional Zig<->script calls, zero-copy where possible.
- **Sandboxing** — memory limits, execution timeouts, API restrictions,
  deterministic mode.
- **Dual syntax** — Lua-style and C-style modes (per the Tree-sitter grammar).
- **JIT-ready** — optional runtime optimization for hot paths.

## Ecosystem

Ghostlang anchors a small toolchain:

- [[Grove]] — Tree-sitter wrapper providing parsing/highlighting.
- `tree-sitter-ghostlang` — the Tree-sitter grammar (dual Lua/C-style).
- [[ghostls]] — the Ghostlang language server (LSP).
- [[GShell]] — a shell that uses Ghostlang for config and plugins.
- [[Grim]] — editor/IDE that embeds Ghostlang for plugins.

> [!key-insight]
> Ghostlang is the connective scripting tissue of the ghost\* desktop/editor
> tools: the same language scripts the shell ([[GShell]]) and the editor
> ([[Grim]]), with shared LSP/grammar infrastructure.

Built on [[Zig]].
