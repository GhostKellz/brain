---
type: concept
title: "Choosing a Local Model for Agents"
created: 2026-06-21
updated: 2026-06-21
tags:
  - llm
  - ollama
  - agents
  - tool-calling
status: seed
related:
  - "[[Ollama]]"
  - "[[Ollama Context Length]]"
  - "[[Local LLM Inference]]"
---

# Choosing a Local Model for Agents

When a local model drives an **agent harness** (Claude Code-style tools, OpenCode,
Cursor-style editors), two independent things determine whether it works at all:

1. **Context** — fixed globally; see [[Ollama Context Length]].
2. **Tool-calling capability** — a *model-choice* problem, covered here.

> [!key-insight]
> Agent harnesses drive everything through structured tool/function calls. A model
> that does unreliable tool-calling — or a reasoning model that wraps output in
> `<think>` tags — breaks the harness's parser regardless of how "smart" it is.
> A weaker model with solid tool-calling beats a stronger one that can't.

## Good drivers (reliable tool-calling)

| Trait | Why it matters |
|-------|----------------|
| Trained for agentic coding / tool use | Emits well-formed tool calls the harness can parse |
| No mandatory `<think>` wrapping | Output goes straight to the parser |
| Mid-size (≈14–30B) | Fits a single GPU at large context |

Examples seen to work well as the main loop: agentic-coding-tuned ~30B models,
Mistral models specifically tuned for tool-calling (~24B), and lighter ~14B
general models with good tool-calling.

## Avoid for *driving* the loop

| Model type | Reason |
|------------|--------|
| Reasoning models (emit `<think>`) | Flaky tool-calling; the think-block confuses parsers |
| Models with weak/no native tool-calling | Harness can't act on their output |
| FIM / completion-oriented code models | Built for inline completion, not agentic steps |
| Vision-language models | Use as an *auxiliary* vision tool, not the main loop |

A reasoning or vision model can still be valuable — just as a *sub-tool* the agent
calls, not as the thing running the agent loop.

## Sizing vs VRAM

A ~30B model at 64k context needs roughly the VRAM of a 32 GB card. On 24 GB
cards, drop to a smaller model or a smaller per-request `num_ctx`. See
[[Local LLM Inference]] and [[Ollama Context Length]] for the VRAM math.

## The two-part rule of thumb

> Local agents work when **context is large enough** *and* the model is a
> **reliable tool-caller**. Get either wrong and it silently fails.

## Related

- [[Ollama Context Length]] — the context half
- [[Local LLM Inference]] — VRAM and GPU offload
- [[Ollama]] — the runtime
