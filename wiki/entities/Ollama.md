---
type: entity
title: "Ollama"
created: 2026-06-21
updated: 2026-06-21
tags:
  - ollama
  - llm
  - local-inference
  - nvidia
status: seed
related:
  - "[[Ollama Service Configuration]]"
  - "[[Ollama Context Length]]"
  - "[[Choosing a Local Model for Agents]]"
  - "[[Local LLM Inference]]"
---

# Ollama

**Ollama** is a local LLM runtime that wraps llama.cpp-style inference behind a
simple HTTP API (`/api/generate`, `/api/chat`, `/api/tags`, `/api/ps`) and a
model registry. It manages model download, GPU offload, KV-cache, and keep-alive,
making it the easiest way to self-host models for agent harnesses and editors.

## Key facts

- **API endpoint**: `http://127.0.0.1:11434` by default (local only).
- **Model store**: blobs + manifests in a models dir (`blobs/`, `manifests/`).
- **GPU build on Arch**: install `ollama-cuda` (not plain `ollama`) for NVIDIA
  offload — see [[Local LLM Inference]].
- **Configured via env vars** applied through a systemd drop-in — see
  [[Ollama Service Configuration]].

## Everyday commands

```bash
ollama list             # installed models
ollama ps               # loaded models + VRAM split + context window
ollama show <model>     # template, params, context window
ollama pull <model>     # download
```

## The two things that break local agents

1. **Context starvation** — the default served context (~4096 tokens) is far too
   small for agent harnesses. Fix globally with `OLLAMA_CONTEXT_LENGTH`; see
   [[Ollama Context Length]].
2. **Weak tool-calling** — many models can't reliably emit structured tool calls,
   or wrap output in `<think>` tags that break parsers. This is a model-choice
   problem; see [[Choosing a Local Model for Agents]].

## Related

- [[Ollama Service Configuration]] — systemd drop-in + env reference
- [[Ollama Context Length]] — the 64k requirement
- [[Choosing a Local Model for Agents]] — model selection for tool use
- [[Local LLM Inference]] — broader context, GPU/VRAM
