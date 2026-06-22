---
type: reference
title: "Ollama Service Configuration"
created: 2026-06-21
updated: 2026-06-21
tags:
  - ollama
  - systemd
  - llm
  - nvidia
status: seed
related:
  - "[[Ollama]]"
  - "[[Ollama Context Length]]"
  - "[[systemd Drop-in Overrides]]"
---

# Ollama Service Configuration

[[Ollama]] is configured almost entirely through environment variables. The clean
way to set them on a systemd system is a **drop-in override** — never edit the
packaged unit, so config survives `ollama-cuda` package updates. See
[[systemd Drop-in Overrides]] for the general pattern.

## The drop-in

`/etc/systemd/system/ollama.service.d/override.conf`:

```ini
[Service]
# Models on a large/fast disk instead of /var/lib/ollama
Environment="OLLAMA_MODELS=/data/ollama"

# Default served context window (agent harnesses need a large one)
Environment="OLLAMA_CONTEXT_LENGTH=65536"

# --- Performance (NVIDIA, large-VRAM card) ---
Environment="OLLAMA_FLASH_ATTENTION=1"     # faster + lower attention VRAM
Environment="OLLAMA_KV_CACHE_TYPE=q8_0"    # 8-bit KV cache; needs flash attn
Environment="OLLAMA_MAX_LOADED_MODELS=2"   # small model resident next to a big one
Environment="OLLAMA_NUM_PARALLEL=2"        # concurrent requests per model
Environment="OLLAMA_KEEP_ALIVE=30m"        # keep models hot
Environment="OLLAMA_HOST=127.0.0.1:11434"  # 0.0.0.0:11434 to expose to LAN
```

## Variable reference

| Variable | Purpose |
|----------|---------|
| `OLLAMA_MODELS` | The models dir itself (holds `blobs/` + `manifests/`), not a parent. |
| `OLLAMA_CONTEXT_LENGTH` | Default served context. Without it, ~4096 — silently breaks agents. See [[Ollama Context Length]]. |
| `OLLAMA_FLASH_ATTENTION` | Faster attention, lower VRAM. **Required** for `q8_0` KV cache. |
| `OLLAMA_KV_CACHE_TYPE` | `q8_0` quantizes the KV cache to 8-bit — roughly halves KV VRAM vs f16, critical at large context. |
| `OLLAMA_MAX_LOADED_MODELS` | Allows e.g. an embedding model to stay resident next to one large model. |
| `OLLAMA_NUM_PARALLEL` | Concurrent requests per loaded model. |
| `OLLAMA_KEEP_ALIVE` | How long an idle model stays in VRAM (default ~5m). |
| `OLLAMA_HOST` | Bind address. `0.0.0.0:11434` exposes to the LAN. |

## Applying changes

```bash
sudo systemctl daemon-reload
sudo systemctl restart ollama
systemctl show ollama -p Environment   # verify EVERY var took, not just the one you changed
```

## The footgun: never overwrite the drop-in wholesale

> [!key-insight]
> A blind `tee > override.conf` that writes only one setting silently drops every
> other line. This is exactly how an `OLLAMA_MODELS` line was once lost, making
> all models appear to "vanish" (the service fell back to an empty default dir).
> Always read the file first, then edit in place, then verify with
> `systemctl show ollama -p Environment`.

## Permissions

The models dir is owned by the `ollama` service user, and any parent must be
world-traversable so the service can reach it:

```bash
sudo chown -R ollama:ollama /data/ollama
# parent (/data) should be drwxrwxr-x
```

## Related

- [[Ollama]] — the runtime
- [[Ollama Context Length]] — why 64k
- [[systemd Drop-in Overrides]] — the general override mechanism
