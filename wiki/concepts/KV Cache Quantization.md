---
type: concept
title: "KV Cache Quantization"
created: 2026-06-21
updated: 2026-06-21
tags:
  - llm
  - vram
  - inference
  - ollama
status: seed
related:
  - "[[Local LLM Inference]]"
  - "[[Ollama Context Length]]"
  - "[[Ollama Service Configuration]]"
---

# KV Cache Quantization

During autoregressive generation, a transformer caches the **key/value tensors**
of every previous token (the "KV cache") so it doesn't recompute attention over
the whole prompt each step. The KV cache lives in VRAM and grows linearly with
context length — at long contexts it can rival or exceed the model weights.

> [!key-insight]
> KV-cache VRAM ≈ proportional to (context length × layers × hidden size × 2).
> Doubling the served context roughly doubles KV-cache VRAM. This is why a model
> that fits at 8k can OOM at 64k.

## Quantizing the cache

Storing KV entries at lower precision shrinks the cache:

| KV cache type | Precision | Relative VRAM | Quality impact |
|---------------|-----------|---------------|----------------|
| `f16` | 16-bit | 1.0× (baseline) | none |
| `q8_0` | 8-bit | ~0.5× | negligible |
| `q4_0` | 4-bit | ~0.25× | noticeable at long ctx |

`q8_0` is the sweet spot: roughly **half** the KV VRAM of f16 with quality loss
small enough to ignore for most workloads. This is what makes large context fit
on consumer cards.

## Flash attention is a prerequisite

In Ollama, quantized KV cache **requires flash attention** to be enabled:

```ini
# /etc/systemd/system/ollama.service.d/override.conf
Environment="OLLAMA_FLASH_ATTENTION=1"
Environment="OLLAMA_KV_CACHE_TYPE=q8_0"
```

Flash attention is also independently worth enabling — it computes attention in a
more memory-efficient, often faster way.

## The practical payoff

The combination of flash attention + `q8_0` KV cache is precisely what lets a
~30B model run at 64k context on a 32 GB GPU. Without it the KV cache is ~2×,
pushing the same workload out of VRAM and into a slow CPU/GPU split. See
[[Ollama Context Length]] for the end-to-end VRAM picture.

## Related

- [[Ollama Context Length]] — where this is applied
- [[Local LLM Inference]] — the broader VRAM budget
- [[Ollama Service Configuration]] — the exact env vars
