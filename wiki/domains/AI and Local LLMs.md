---
type: domain
title: "AI and Local LLMs"
created: 2026-06-21
updated: 2026-06-21
tags:
  - domain
  - ai
  - llm
status: developing
related:
  - "[[Linux and Systems]]"
  - "[[Programming Languages]]"
---

# AI and Local LLMs

Local-first inference on the RTX 5090 via [[Ollama]] (interactive) and [[vLLM]]
(high-throughput serving), plus the agentic tooling built on top of them.

## Inference engines
[[Ollama]] · [[Ollama Service Configuration]] · [[Ollama Context Length]] ·
[[vLLM]] · [[Local LLM Inference]]

## Tuning & hardware
[[KV Cache Quantization]] · [[Choosing a Local Model for Agents]] ·
[[NVIDIA Container Toolkit]] · [[CUDA Delivery Paths]]

## Agentic tooling
[[Jarvis]] — local-first AI orchestration/memory runtime.
[[hermes-agent]] — self-improving, model-agnostic autonomous agent (skills +
bounded memory; point it at the local Ollama/vLLM endpoint).
[[openshell]] — NVIDIA sandboxed, policy-governed runtimes for running those agents
safely (egress control + credential isolation + inference routing).

> [!key-insight]
> The two defaults that silently break agent use of [[Ollama]]: the 4096-token
> context window ([[Ollama Context Length]]) and model choice
> ([[Choosing a Local Model for Agents]]).
