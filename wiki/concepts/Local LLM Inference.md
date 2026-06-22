---
type: concept
title: "Local LLM Inference"
created: 2026-06-21
updated: 2026-06-21
tags:
  - llm
  - inference
  - nvidia
  - vram
status: seed
related:
  - "[[Ollama]]"
  - "[[KV Cache Quantization]]"
  - "[[Choosing a Local Model for Agents]]"
  - "[[NVIDIA Container Toolkit]]"
  - "[[CUDA Delivery Paths]]"
---

# Local LLM Inference

Running large language models on your own hardware instead of a hosted API.
Trade-offs: full privacy/control and no per-token cost, against being bounded by
your GPU's VRAM and compute.

## What bounds you: VRAM

The dominant constraint is **GPU VRAM**, which must hold two things at once:

1. **Model weights** — size ≈ parameters × bytes-per-weight. Quantization
   (Q4/Q5/Q8) trades a little quality for big size reductions; a 30B model at
   Q4 is ~18 GB on disk and a similar order in VRAM.
2. **KV cache** — grows with context length; see [[KV Cache Quantization]].

> [!key-insight]
> Fitting a model is not "weights ≤ VRAM" — it's "weights + KV cache at your
> served context ≤ VRAM". A model that loads fine at 4k can OOM at 64k purely
> from KV-cache growth.

Rough guide (consumer NVIDIA):

| Card VRAM | Comfortable |
|-----------|-------------|
| 32 GB | a ~30B Q4 model at 64k context (with q8_0 KV cache) |
| 24 GB | a ~30B model at short context, or a smaller model at long context |
| 16 GB | ~14B and below, or heavy quantization |

## GPU offload on Arch

For NVIDIA, install the CUDA-enabled runtime build (e.g. `ollama-cuda`, not plain
`ollama`). Verify the model is actually on the GPU:

```bash
journalctl -u ollama -n 40 | grep -iE 'gpu|cuda|inference compute'
ollama ps     # want 100% GPU, not a CPU/GPU split
```

A CPU/GPU split means it didn't all fit — reduce model size, context, or enable
[[KV Cache Quantization]].

## Three ways to deliver the GPU

Inference can run on bare metal, inside a passthrough VM, or in a container — all
converging on the same CUDA stack. See [[CUDA Delivery Paths]] and
[[NVIDIA Container Toolkit]].

## Two failure modes for agent use

1. Context too small — silent flailing. See [[Ollama Context Length]].
2. Weak tool-calling — broken harness. See [[Choosing a Local Model for Agents]].

## Related

- [[Ollama]] — the easiest local runtime
- [[KV Cache Quantization]] — making long context fit
- [[CUDA Delivery Paths]] — native vs VFIO vs container
