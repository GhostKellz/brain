---
type: meta
title: "Hot Cache"
aliases:
  - "Hot Cache"
created: 2026-06-21
updated: 2026-06-21
tags:
  - meta
status: developing
related:
  - "[[Wiki Index]]"
  - "[[Brain Overview]]"
---

# Recent Context

## Last Updated
2026-06-21 — Initial seed of the brain. Four parallel extraction passes plus a
hand-built backbone.

## Key Recent Facts
- The brain (`~/brain` → `github.com/ghostkellz/brain`, public) is now seeded with
  ~130 cross-linked pages across 8 domains.
- Unifying thesis captured: local-first infra, **Rust for apps / [[Zig]] for engines**.
- Sources mined: `~/arch` (public-safe subset), `/data/projects` (85 repos),
  the three web properties, and the (sanitized) CK Cheatsheet.

## Recent Changes
- Created backbone: [[Brain Overview]], [[Wiki Index]], 8 domain hubs,
  [[GhostKellz]], [[CK Technology LLC]].
- Seeded entities (47), concepts (38), references (39), sources (3).
- Resolved a filename collision: split Tailscale into [[Tailscale]] (entity) +
  [[Tailscale Operations]] (runbook).

## Tiering reminders
- PUBLIC vault. `heimdall-stack`/`security` specifics excluded → private tier.
- Cheatsheet was sanitized: client domains/IPs/keys removed or placeholdered.
- Client docs stay in Hudu, never here.

## Active Threads
- Done: seed committed; MCPVault filesystem MCP wired for both Claude Code and
  Codex; local hybrid retrieval (BM25 + ollama `nomic-embed-text` rerank) built.
- Tiering update: one vault only — public content is tracked/pushed; private notes
  live in gitignored `private/` paths, local-only (no separate GitLab vault).
- Pending later phases: MemPalace eval, Quartz publish to `brain.ckelley.dev`,
  optional Graphify code-graph visuals. Eventually capture config in
  `~/arch/dotfiles/claude`.
