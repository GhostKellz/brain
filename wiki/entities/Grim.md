---
type: entity
title: "Grim"
created: 2026-06-21
updated: 2026-06-21
tags:
  - editor
  - ide
  - zig
  - vim
status: seed
entity_type: repository
language: Zig
repo_status: active
purpose: "Lightweight Zig-powered IDE/editor with Vim soul and modern features"
related:
  - "[[Zig]]"
  - "[[Grove]]"
  - "[[Ghostlang]]"
  - "[[ghostls]]"
  - "[[Phantom]]"
  - "[[GhostKellz]]"
---

# Grim

A lightweight, [[Zig]]-powered IDE/editor with a "Vim soul and modern brains."
Part of the [[Ghost Ecosystem]]; by [[GhostKellz]].

## Stack & integrations

- **Parsing/highlighting**: [[Grove]] (Tree-sitter wrapper).
- **Scripting/plugins**: [[Ghostlang]].
- **Language intelligence**: [[ghostls]] and other LSPs.
- Shares TUI primitives with the [[Phantom]] framework.

Grim is the editor that ties the Ghostlang toolchain together — it is the
original consumer that [[Grove]] was built for and a primary target of
[[ghostls]].

Built on [[Zig]].
