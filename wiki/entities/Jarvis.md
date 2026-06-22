---
type: entity
title: "Jarvis"
created: 2026-06-21
updated: 2026-06-21
tags:
  - ai
  - orchestration
  - rust
  - cli
  - mcp
status: seed
entity_type: repository
language: Rust
repo_status: active
purpose: "Local-first AI orchestration and memory runtime that connects Claude Code, Codex, OpenCode, Gemini CLI and local models"
related:
  - "[[Rust]]"
  - "[[zqlite]]"
  - "[[Zeke]]"
  - "[[GhostKellz]]"
---

# Jarvis

A **local-first orchestration layer** that sits above AI coding tools. Jarvis
does not replace Claude Code, Codex, OpenCode, or Gemini CLI — it connects them,
remembers context across sessions, and routes requests to the right provider.
Authored by [[GhostKellz]], MIT-licensed, [[Rust]]-based.

## Language & stack

- **Language**: Rust, async on Tokio, CLI via Clap.
- **Storage**: SQLite with FTS5 for memory + search; hybrid search fuses FTS5
  keyword matches with local embeddings by rank.
- **Local models**: Ollama (default `llama3.1:8b` for summarization,
  `nomic-embed-text` for embeddings); degrades cleanly to keyword-only when
  Ollama is offline.
- **Platform**: Arch Linux only for v1 (targets Neovim / terminal power users).

## Architecture

- Rust CLI plus a background daemon.
- SQLite + FTS memory/search engine.
- Hook integrations (Claude Code) and an MCP server.
- Execution adapters that drive Claude, Codex, Gemini, and OpenCode as CLI
  subprocesses.
- Intent-based provider routing with availability-aware fallback ("The Foreman").
- Voice loop layered over the `run` core.

## Notable features

- **Session tracking** — active tool, repo + branch awareness, lifecycle.
- **Memory engine** — project memory, decisions/tasks, hybrid keyword+semantic
  search (`jarvis memory search`).
- **Resume / handoff** — `jarvis resume` reconstructs what you were doing and
  suggests the next prompt, across any tool-to-tool transition.
- **The Foreman** — `jarvis run "..."` assembles context, picks a provider by
  `--intent`, and executes end to end; `jarvis route`, `jarvis providers`.
- **Voice Foreman** — mic -> whisper.cpp (STT) -> Foreman -> Piper (TTS) over
  PipeWire; VAD-endpointed continuous loop gated on a "Hey Jarvis" wake word via
  whisper keyword-spotting (no separate wake-word model).
- **MCP server** — exposes context to MCP clients over JSON-RPC stdio with tools
  `jarvis_get_context`, `jarvis_search_memory`, `jarvis_add_memory`,
  `jarvis_list_tasks`.
- **Auto-summarize** — optional `[daemon]` opt-in that summarizes a session on
  end via the stale-session reaper or the `SessionEnd` hook, non-blocking.

## Status

Early-stage; built for real workflows first. Active development with a detailed
changelog.

> [!key-insight]
> Jarvis's philosophy is "not the AI" — it is the memory, the context, and the
> orchestrator, leaving the underlying tools as the brains and execution engines.

## Relationships

- Built on [[Rust]].
- Overlaps conceptually with [[Zeke]] (local-first AI routing/caching/transport
  for Rust agents) — both deal with multi-provider AI routing, Jarvis at the
  orchestration/memory layer.
- Its SQLite-backed memory is the kind of workload [[zqlite]] targets in the Zig
  half of the ecosystem.
