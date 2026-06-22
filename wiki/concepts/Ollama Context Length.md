---
type: concept
title: "Ollama Context Length"
created: 2026-06-21
updated: 2026-06-21
tags:
  - ollama
  - llm
  - agents
  - vram
status: seed
related:
  - "[[Ollama]]"
  - "[[Ollama Service Configuration]]"
  - "[[Choosing a Local Model for Agents]]"
  - "[[KV Cache Quantization]]"
---

# Ollama Context Length

> [!key-insight]
> Ollama's **default served context is ~4096 tokens**. Agent harnesses burn
> 10–20k tokens on system prompt + tool schemas *before the user says anything*.
> At 4096 the model never sees the full instructions, so it flails, ignores
> tools, or loops — **with no error message**. This is the single most common
> reason local models "don't work" in agent tools.

## The fix

Serve a large context globally via [[Ollama Service Configuration]]:

```ini
# /etc/systemd/system/ollama.service.d/override.conf
Environment="OLLAMA_CONTEXT_LENGTH=65536"
```

`65536` (64 Ki) is a clean power of two that clears typical harness floors (some
harnesses hard-reject models serving < 64k) with headroom.

A subtlety: the **models** (qwen3, etc.) already support large contexts. Ollama
just won't *serve* them that large until told to. Setting the model's max context
is not enough — `OLLAMA_CONTEXT_LENGTH` is the served default.

## The VRAM cost

Bigger context = bigger KV cache resident in VRAM. This is why
[[KV Cache Quantization]] (`OLLAMA_KV_CACHE_TYPE=q8_0` + flash attention) matters:
together they roughly halve KV-cache VRAM, which is what makes 64k fit on a 32 GB
card.

Rough figure: a ~30B Q4 model at 64k context is on the order of ~25 GB resident —
fits a 32 GB card with headroom, but is tight-to-impossible on 24 GB cards.

## Verify what's actually served

```bash
# warm a model
curl -s http://localhost:11434/api/generate \
  -d '{"model":"<model>","prompt":"hi","stream":false,"keep_alive":"5m"}' >/dev/null

# check the loaded context window
curl -s http://localhost:11434/api/ps | python3 -c \
  "import sys,json; [print(m['name'], m.get('context_length')) for m in json.load(sys.stdin)['models']]"
```

If `context_length` shows `4096`, the env var didn't take — re-check
`systemctl show ollama -p Environment` and that the service restarted.

## Per-request override

`OLLAMA_CONTEXT_LENGTH` is the default; a client can request a smaller window per
call (useful on smaller-VRAM machines):

```json
{ "model": "<model>", "options": { "num_ctx": 16384 } }
```

## Related

- [[Ollama Service Configuration]] — where the env var lives
- [[KV Cache Quantization]] — making large context fit in VRAM
- [[Choosing a Local Model for Agents]] — the other half of "local agents work"
