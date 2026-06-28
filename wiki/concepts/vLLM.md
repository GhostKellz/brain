---
type: concept
title: "vLLM"
created: 2026-06-28
updated: 2026-06-28
aliases:
  - "PagedAttention"
  - "Continuous Batching"
  - "OpenAI-Compatible Server"
tags:
  - llm
  - inference
  - vllm
  - nvidia
  - vram
  - throughput
status: developing
related:
  - "[[Local LLM Inference]]"
  - "[[Ollama Context Length]]"
  - "[[KV Cache Quantization]]"
  - "[[CUDA Delivery Paths]]"
  - "[[Choosing a Local Model for Agents]]"
---

# vLLM

**vLLM** is a high-throughput inference and serving engine for LLMs. Where
[[Ollama]] optimizes for *one machine, one user, zero setup*, vLLM optimizes for
*serving many concurrent requests at maximum tokens/sec* — the engine you reach
for when a single GPU (or a cluster) has to feed an agent fleet, a batch job, or
an internal API rather than one interactive chat.

> [!key-insight]
> Ollama and vLLM solve different problems. Ollama is the latency-optimized
> *daily driver* (load a GGUF, chat, done). vLLM is the throughput-optimized
> *server* — its whole architecture (PagedAttention + continuous batching) exists
> to keep the GPU saturated across **many simultaneous requests**, serving 3–5×
> the traffic of a naive PyTorch loop on the same card. Pick vLLM when concurrency
> and tokens/sec matter; pick Ollama when it's just you.

## The two innovations

vLLM's throughput comes from two compounding ideas:

### PagedAttention

Classic inference allocates one contiguous KV-cache slab per request, sized to
the *max* context — most of it sits empty, and fragmentation wastes the rest.
PagedAttention treats the [[KV Cache Quantization|KV cache]] like an OS virtual-
memory system: it splits the cache into fixed **blocks** mapped through a block
table, so memory is allocated on demand in small pages instead of one big
reservation.

> [!key-insight]
> PagedAttention cuts KV-cache fragmentation by roughly **4×**. That reclaimed
> VRAM becomes either more concurrent requests or more context per request — the
> same lever [[KV Cache Quantization]] pulls, but structural rather than numeric.
> The two stack: paged blocks *and* an FP8 cache dtype.

### Continuous batching

A naive server batches requests, runs them to completion, then starts the next
batch — short requests wait on long ones (head-of-line blocking). vLLM does
**continuous (in-flight) batching**: as soon as any sequence in the batch
finishes, its slot is freed and a queued request takes its place that same step.
Combined with **chunked prefill** (interleaving prompt-processing with
generation), the GPU never idles waiting for a batch boundary.

## The OpenAI-compatible server

vLLM's most practical feature for a homelab is that it speaks the **OpenAI API**.
Anything that talks to OpenAI (agent harnesses, SDKs, editors) points at vLLM by
changing the base URL — no client rewrite.

```bash
# Serve a model with an OpenAI-compatible endpoint
vllm serve <model-id> \
  --tensor-parallel-size 2 \
  --max-model-len 65536 \
  --kv-cache-dtype fp8

# Call it like OpenAI (just a different base URL)
curl -s http://localhost:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"model":"<model-id>","messages":[{"role":"user","content":"hi"}]}'
```

This is what lets every local endpoint — Ollama for interactive, vLLM for
throughput — present the **same OpenAI surface**, so an agent's
[[Choosing a Local Model for Agents|harness]] doesn't care which is behind it.

## Multi-GPU & distributed parallelism

vLLM scales past one card with four parallelism axes that can combine:

| Axis | Flag | What it splits | When |
|------|------|----------------|------|
| **Tensor** (TP) | `--tensor-parallel-size N` | Each layer's weights *across* GPUs | Model too big for one card; needs fast interconnect |
| **Pipeline** (PP) | `--pipeline-parallel-size N` | Whole layers into stages across nodes | Scale across hosts / slower links |
| **Data** | replicas | Whole-model copies behind a router | More throughput, model already fits |
| **Expert** (EP) | MoE configs | Mixture-of-Experts experts across GPUs | Large MoE models |

> [!key-insight]
> Tensor parallelism shards a model that won't fit one card across several,
> letting heterogeneous GPUs serve a single large model cooperatively — e.g. a
> native card plus a [[CUDA Delivery Paths|VFIO-passthrough]] card on the same
> host. TP wants high intra-node bandwidth (NVLink/PCIe); reach for **pipeline**
> parallelism when you're spanning separate machines over a network link rather
> than a bus.

A representative homelab topology: a primary GPU runs the interactive
[[Ollama]] daily driver, while vLLM tensor-parallel shards a larger model across
that GPU and a passed-through second card; for high-throughput batch serving it
scales out further to clustered accelerators over a fast (10 GbE+) link — every
endpoint still speaking the same OpenAI API behind a fallback router.

> [!note] In Docker, multi-GPU TP needs `--ipc=host` (or a large `--shm-size`) so
> the per-GPU worker processes can share memory; see [[CUDA Delivery Paths]] and
> [[NVIDIA Container Toolkit]] for getting the driver into the container.

## Quantization

vLLM supports weight quantization (**AWQ**, **GPTQ**, INT8, **FP8**) *and*
separate **KV-cache** quantization:

| Scheme | Precision | VRAM vs FP16 | Notes |
|--------|-----------|--------------|-------|
| **FP8** (W8A8) | 8-bit weights+activations | ~0.5× | Native on Hopper/Ada/Blackwell; ~1.6× throughput, tiny accuracy loss; no calibration for dynamic FP8 |
| **AWQ / GPTQ** | 4-bit weights | ~0.25× | Calibration step; AWQ common for vLLM; small quality regression |
| **INT8** | 8-bit | ~0.5× | Good on Ampere (A100) which lacks FP8 hardware |
| **FP8 KV cache** | `--kv-cache-dtype fp8` | KV ~0.5× | Doubles tokens the cache pool holds (mirrors [[KV Cache Quantization]]) |

> [!key-insight]
> FP8 is the 2026 sweet spot on modern NVIDIA: ~2× memory reduction and a
> throughput bump with negligible quality loss, and dynamic FP8 needs **no
> calibration data** (`--quantization fp8`). On older Ampere cards without FP8
> tensor cores, fall back to INT8 or 4-bit AWQ/GPTQ. The same FP8-KV-cache trick
> that doubles servable context in [[KV Cache Quantization]] applies here via
> `--kv-cache-dtype fp8`.

## vLLM vs Ollama

| | Ollama | vLLM |
|--|--------|------|
| Optimized for | Single-user latency, zero setup | Multi-request throughput |
| Model format | GGUF (llama.cpp) | HF safetensors (+ AWQ/GPTQ/FP8) |
| Concurrency | Limited parallelism | Continuous batching, built for it |
| Multi-GPU | Basic | Tensor/pipeline/data/expert parallel |
| API | Native + OpenAI-compat | OpenAI-compatible server |
| Setup | One binary, trivial | Heavier (CUDA/PyTorch toolchain) |
| Best when | It's just you, interactive | An API/agent fleet, batch jobs |

The two are complementary, not rivals: Ollama for the interactive daily driver,
vLLM when something needs to serve **many** callers fast. See
[[Local LLM Inference]] for the shared VRAM constraints both live under.

## Pros / trade-offs

**Pros**
- Best-in-class throughput for concurrent serving (PagedAttention + batching).
- Drop-in OpenAI API — existing clients/harnesses just change base URL.
- First-class multi-GPU/distributed and broad quantization (FP8/AWQ/GPTQ).

**Trade-offs**
- Heavier setup than Ollama (CUDA/PyTorch stack; usually run via Docker).
- Defaults shift between minor versions (chunked-prefill, prefix-caching) — **pin
  the image tag**, never `:latest`, for reproducible deploys.
- Overkill for a single interactive user — that's exactly where Ollama wins.

## Related

- [[Local LLM Inference]] — the shared VRAM budget both engines obey
- [[KV Cache Quantization]] — the cache-precision lever vLLM also exposes (`--kv-cache-dtype`)
- [[CUDA Delivery Paths]] — native vs VFIO vs container for the GPUs vLLM shards
- [[Ollama Context Length]] · [[Choosing a Local Model for Agents]] — the agent-serving counterparts
