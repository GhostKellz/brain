---
type: entity
title: Jarvis
created: 2026-06-21
updated: 2026-06-28
tags:
  - ai
  - orchestration
  - rust
  - cli
  - mcp
  - memory
  - voice
status: active
entity_type: repository
language: Rust
repo_status: active
purpose: Local-first AI orchestration and memory runtime that unifies Claude
  Code, Codex, OpenCode, Gemini CLI and local models into one workflow
related:
  - "[[Rust]]"
  - "[[Ollama]]"
  - "[[Local LLM Inference]]"
  - "[[Zeke]]"
  - "[[zqlite]]"
  - "[[GhostKellz]]"
---

# Jarvis

A **local-first AI orchestration and memory runtime** that sits above AI coding
tools. Jarvis does not replace Claude Code, Codex, OpenCode, Gemini CLI, or local
[[Ollama]] models — it unifies them into one workflow, remembers context across
sessions, and routes work to the right runner. Authored by [[GhostKellz]],
MIT-licensed, [[Rust]]-based.

## Language & stack

- **Language**: [[Rust]] (2024 edition), async on Tokio.
- **Storage**: SQLite (WAL, pooled) with FTS5 for memory and search.
- **Local models**: [[Ollama]] for summarization and `nomic-embed-text`
  embeddings; see [[Local LLM Inference]].

## Hybrid memory engine

Memory fuses SQLite FTS5 keyword matching with local embeddings (Ollama
`nomic-embed-text`) by rank. When Ollama is offline it degrades gracefully to
keyword-only search, so memory always works. Stores project memory, decisions,
and tasks.

## Capabilities

- **Session tracking** — detects active repo, branch, and tool; tracks session
  lifecycle.
- **Resume / handoff** — reconstructs what you were doing and suggests the next
  prompt across any tool-to-tool transition.
- **Local context generation** — produces summaries and briefings from session
  history.
- **Foreman** — intent-based runner: routes a request by intent, falls back in
  an availability-aware way, and supports dry-run.
- **Voice Foreman** — mic to whisper.cpp (STT) to Foreman to Piper (TTS), with
  VAD endpointing and wake-word gating for a continuous voice loop.
- **MCP server** — exposes context to MCP clients over JSON-RPC stdio with tools
  `jarvis_get_context`, `jarvis_search_memory`, `jarvis_add_memory`, and
  `jarvis_list_tasks`.
- **TUI** — ratatui terminal interface.
- **Visualization** — `jarvis viz` renders execution-flow diagrams.
- **Resilience** — backoff, circuit breaker, and tool-call validation/repair.

## Status

v0.1 — early but usable for real workflows. It has grown a lot yet remains in its
infancy; active development. MIT-licensed.

> [!key-insight]
> Jarvis's philosophy is "not the AI" — it is the memory, the context, and the
> orchestrator, leaving the underlying tools as the brains and execution engines.

## Relationships

- Built on [[Rust]]; uses [[Ollama]] for [[Local LLM Inference]].
- Overlaps conceptually with [[Zeke]] (local-first AI routing/caching/transport
  for Rust agents) — both deal with multi-provider AI routing, Jarvis at the
  orchestration/memory layer.
- Its SQLite-backed memory is the kind of workload [[zqlite]] targets in the Zig
  half of the ecosystem.
