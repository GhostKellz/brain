---
type: entity
title: "Grove"
created: 2026-06-21
updated: 2026-06-21
tags:
  - parsing
  - tree-sitter
  - zig
status: seed
entity_type: repository
language: Zig
repo_status: active
purpose: "High-performance Tree-sitter wrapper for Zig providing safe, ergonomic syntax highlighting and parsing"
related:
  - "[[Zig]]"
  - "[[Ghostlang]]"
  - "[[Grim]]"
  - "[[ghostls]]"
  - "[[Phantom]]"
  - "[[GhostKellz]]"
---

# Grove

A modern [[Zig]] wrapper around the Tree-sitter parsing library, built to give
the [[Grim]] editor safe, ergonomic syntax highlighting and incremental parsing.

## Focus

- **Safety** — RAII resource management with no undefined behavior on moved
  trees.
- **Performance** — zero-copy rope integration and incremental parsing.

## Role in the ecosystem

Grove is the parsing backbone for the ghost\* editor stack: it powers syntax
highlighting in [[ghostls]] (the Ghostlang LSP) and [[Grim]], and is one of the
advanced integrations available in the [[Phantom]] TUI framework. It parses
[[Ghostlang]] via `tree-sitter-ghostlang`, among other grammars.

Built on [[Zig]]; by [[GhostKellz]].
