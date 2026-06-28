---
type: domain
title: "Programming Languages"
created: 2026-06-21
updated: 2026-06-21
tags:
  - domain
  - programming
  - zig
  - rust
  - go
  - python
  - javascript
status: developing
related:
  - "[[DevOps and Homelab]]"
  - "[[AI and Local LLMs]]"
---

# Programming Languages

The two-language strategy across the ecosystem: **Rust for applications, Zig for
engines/runtimes**. The general-purpose languages — [[Go]], [[Python]],
[[JavaScript]] — cover cloud-native tooling, scripting/data/ML, and the web.

## Zig
[[Zig]] — language overview and why it underpins the engines.
Built in Zig: [[zqlite]] · [[Ghostlang]] · [[Grove]] · [[GShell]] · [[Grim]] ·
[[ghostls]] · [[Phantom]] · [[zcrypto]] · [[zquic]] · [[zsync]]

## Rust
[[Rust]] — language overview and why it covers the application/service layer.
[[Rust Ownership and Borrowing]] · [[Cargo Workflow]] (rustup/rustc/cargo)
Built in Rust: [[Jarvis]] · [[nvcontrol]] · [[ghostctl]] · [[GhostChain]] ·
[[GhostWire]] · [[Bolt]] · [[ghostbrew]] · [[Zeke]] · [[GhostCP]] ·
[[ghostport]] · [[ghostcrate]] · [[reaper]] · [[GhostView]]

## Go
[[Go]] — cloud-native / CLI / ops-tooling language: goroutines, single static
binaries, the `cobra`/`viper` + Charm (`bubbletea`/`bubbles`/`lipgloss`) stacks.

## Python
[[Python]] — scripting, data/ML, and glue: venvs on Linux, `pip`/`pipx`/`uv`,
Flask/FastAPI, and the must-have library set. Drives the vault's retrieval scripts.

## JavaScript
[[JavaScript]] — the web language + its runtimes: Node.js, Bun, Deno; npm and
how dependencies/lockfiles work; TypeScript.

## Scripting
[[Ghostlang]] (.gza) — embeddable Lua-replacement scripting engine, with
[[ghostls]] (language server) and [[GShell]] (scriptable shell).
