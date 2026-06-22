---
type: entity
title: "ghostls"
created: 2026-06-21
updated: 2026-06-21
tags:
  - lsp
  - zig
  - ghostlang
  - tooling
status: seed
entity_type: repository
language: Zig
repo_status: active
purpose: "Native Zig language server for Ghostlang, powered by Grove"
related:
  - "[[Zig]]"
  - "[[Ghostlang]]"
  - "[[Grove]]"
  - "[[Grim]]"
  - "[[GhostKellz]]"
---

# ghostls

The official **Language Server** for [[Ghostlang]], built in [[Zig]] for speed
and tight ecosystem integration. By [[GhostKellz]].

## Features

- Syntax highlighting, document symbols, navigation, completions.
- Powered by [[Grove]] (the Tree-sitter engine).
- Designed to plug into [[Grim]], VSCode, and Neovim.

ghostls completes the Ghostlang toolchain (engine -> grammar -> LSP -> editor)
alongside [[Grove]] and `tree-sitter-ghostlang`.

Built on [[Zig]].
