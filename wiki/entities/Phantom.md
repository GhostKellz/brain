---
type: entity
title: "Phantom"
created: 2026-06-21
updated: 2026-06-21
tags:
  - tui
  - zig
  - framework
status: seed
entity_type: repository
language: Zig
repo_status: active
purpose: "Terminal user interface framework for Zig, inspired by Ratatui"
related:
  - "[[Zig]]"
  - "[[Grove]]"
  - "[[Grim]]"
  - "[[GhostKellz]]"
---

# Phantom

A terminal user interface (TUI) framework for [[Zig]], inspired by Ratatui. By
[[GhostKellz]].

## Strengths

- App-style TUIs built around `App`, core widgets, and a `layout.engine`.
- Theme-driven interfaces with manifest loading and live refresh.
- Dashboards, monitoring views, and async-backed data presentation.
- Unicode-aware rendering, transitions, and a configurable event loop.
- Advanced integrations: [[Grove]] syntax highlighting and PTY-backed terminal
  sessions.

Phantom is the Zig analogue to ratatui (which the Rust-side [[nvcontrol]] TUI
uses), and supplies TUI primitives across ghost\* tooling.

Built on [[Zig]].
