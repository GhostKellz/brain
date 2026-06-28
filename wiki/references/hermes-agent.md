---
type: reference
title: "hermes-agent"
created: 2026-06-28
updated: 2026-06-28
aliases:
  - "Hermes Agent"
  - "Hermes"
  - "Nous Hermes Agent"
tags:
  - hermes
  - ai-agent
  - llm
  - agentic
  - ollama
  - mcp
status: developing
related:
  - "[[Local LLM Inference]]"
  - "[[Ollama]]"
  - "[[Choosing a Local Model for Agents]]"
  - "[[vLLM]]"
  - "[[Jarvis]]"
---

# hermes-agent

**Hermes Agent** (Nous Research, MIT, Python 3.11+) is a **self-improving,
model-agnostic autonomous agent** you run on your own box, a VPS, or a GPU node.
Unlike a stateless chat UI it **persists across sessions**: it writes reusable
**skills** from experience, keeps a **bounded memory** that resists context bloat,
and builds a model of you over time. Swap Claude ↔ GPT ↔ a local model with
`hermes model` and **all learned state survives** — because the value lives in the
harness, not the model.

> [!key-insight]
> The thesis is **harness engineering: the wrapper matters more than the model.**
> Memory, skills, constraints, and orchestration live in `~/.hermes/` — *outside*
> the model — so you can point it at a cheap local [[Ollama]] endpoint or frontier
> Claude without losing anything. That's the opposite of a vendor-locked assistant,
> and why it pairs naturally with a [[Local LLM Inference|local-first inference]]
> stack.

## Mental model

Two entry points, **one** agent instance (same memory/skills/cron):

- **Terminal** — `hermes` (classic CLI) or `hermes --tui` (recommended TUI).
- **Gateway** — a message broker that bridges Telegram/Discord/Slack/Signal/Email/
  Home Assistant/etc. into the same agent.

State lives under `~/.hermes/`:

| Path | What |
|------|------|
| `config.yaml` | Non-secret config (model, providers, toolsets, memory, delegation, gateway, MCP, cron) |
| `.env` | Secrets (API keys, bot tokens) — `chmod 600` |
| `memories/MEMORY.md` | Active working context (~2.2 KB cap) |
| `memories/USER.md` | User facts/preferences (~1.4 KB cap) |
| `state.db` | FTS5 full-text session archive (unbounded, searchable) |
| `SOUL.md` | Persona / system prompt |
| `skills/` | Installed + agent-generated skills |
| `cron/` | Scheduled jobs |

## The three pillars

### Bounded memory

Three layers with **deliberate size caps** that force consolidation: a small
working `MEMORY.md`, a small `USER.md`, and an **unbounded FTS5 archive** in
`state.db`. You don't lose old context — you shift from "in-context" to
"search-then-recall", which is what keeps the prompt from rotting.

### Self-generated skills

After a task (~every few tool calls) the agent introspects — *did it succeed, was
it efficient, is it repeatable?* — and if so writes a markdown **skill** to
`~/.hermes/skills/`. Later similar requests trigger that skill, so it gets faster
and more reliable. An **autonomous curator** consolidates overlapping skills and
archives stale ones. Trust tiers: builtin → official → trusted → community.

### Model-agnostic providers

18+ providers from day one (Anthropic, OpenAI, OpenRouter, Gemini, DeepSeek,
Azure, Bedrock, NVIDIA NIM, …) **plus local** ([[Ollama]], LM Studio, [[vLLM]],
any OpenAI-compatible `/v1`). **Fallback chains** go primary → secondary → local
(zero-cost last resort). Switching models preserves state.

## Install & run

```bash
# Linux/macOS/WSL2
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash

hermes setup          # wizard: provider, tools, gateway
hermes setup --portal # fast path via Nous Portal OAuth
hermes --tui          # start the TUI
hermes --continue     # resume last session
hermes model          # swap model/provider (state preserved)
hermes doctor         # health check
```

> [!warning]
> Requires **Python 3.11–3.13** (Arch's 3.14 is too new — install via `uv`) and a
> model with **≥ 64k context** (smaller contexts break tool-calling reliability).
> It's **zero-knowledge for skills/memory but not for API keys** — secrets sit in
> `~/.hermes/.env`; lock it down.

## Capabilities worth knowing

- **40+ tools**: code execution, browser (CDP), web search, file ops (with approval
  gates), image gen/vision, memory, delegation, `send_message`.
- **Delegation**: spawn isolated subagents (`delegate_task`), up to 3 concurrent;
  child tool calls never pollute the parent context; children **auto-deny**
  destructive commands.
- **Cron**: natural-language scheduled jobs ("every morning, summarize GitHub and
  send to Telegram"); runs as a daemon / systemd service (`hermes gateway install`).
- **MCP**: connect external Model Context Protocol servers (HTTP or stdio) and
  also `hermes mcp serve` to expose Hermes itself as an MCP server to other agents/
  editors. Pairs with editor pair-programming via ACP.
- **Terminal backends**: local, Docker-isolated, SSH (run on a [[Proxmox]] VM),
  plus serverless (Modal/Daytona). **Checkpoints** snapshot the filesystem before
  destructive ops for rollback.
- **Voice**: local Faster-Whisper STT + Edge/OpenAI/ElevenLabs TTS over the gateway.

## Where it fits here

- Runs against the **local-first** GPU stack — point `custom_providers` at the
  [[Ollama]] or [[vLLM]] endpoint instead of paying per-token.
- Complements [[Jarvis]] as the general agent harness; model choice guided by
  [[Choosing a Local Model for Agents]].
- The "harness > model" design is why it's filed as durable infrastructure, not a
  throwaway chat client.

## Pros / trade-offs

**Pros**
- Provider-agnostic with local models as first-class; no lock-in.
- Persistent, self-improving (skills + bounded memory + searchable archive).
- One instance across CLI + 14 messaging platforms; built-in cron + delegation.
- MCP both ways; safe-by-default (approval gates, checkpoints, child auto-deny).

**Trade-offs**
- Another self-hosted service to run/patch; secrets management is on you.
- Needs a ≥64k-context model — tiny local models won't drive it reliably.
- Breadth = surface area; lock down tools/gateway profiles for exposed instances.

## Related

- [[Local LLM Inference]] · [[Ollama]] · [[vLLM]] — the backends Hermes points at
- [[Choosing a Local Model for Agents]] — pick a model with enough context
- [[Jarvis]] — companion agent work in the local-first stack
