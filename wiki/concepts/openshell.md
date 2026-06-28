---
type: concept
title: "openshell"
created: 2026-06-28
updated: 2026-06-28
aliases:
  - "OpenShell"
  - "NVIDIA OpenShell"
tags:
  - openshell
  - nvidia
  - ai-agent
  - sandbox
  - security
  - llm
status: seed
related:
  - "[[hermes-agent]]"
  - "[[Docker]]"
  - "[[Local LLM Inference]]"
  - "[[Zero Trust Networking]]"
  - "[[NVIDIA Container Toolkit]]"
---

# openshell

**openshell** (NVIDIA) provides **sandboxed, policy-governed runtimes for
autonomous AI agents**. A gateway control plane manages sandbox lifecycle; each
agent runs in its own container with **policy-enforced egress** and **credential
isolation**. It's the safety harness you wrap around an agent that can run code and
hit the network — so you can let Claude/Codex/Copilot/OpenCode loose without
trusting them with your keys or your network.

> [!key-insight]
> The interesting bit is the **three-outcome egress model**: every outbound
> connection from the sandbox is intercepted and either **allowed**, **routed for
> inference** (the agent's own API creds are *stripped* and managed backend creds
> *injected*), or **denied and logged**. Credentials are injected as env vars at
> launch and **never written to the sandbox filesystem** — a clean
> [[Zero Trust Networking|zero-trust]] boundary for code you don't fully trust.

## Why it ties into the AI stack

Autonomous agents (like [[hermes-agent]]) want to execute code, browse, and call
LLM APIs. That's exactly the capability you *don't* want unsandboxed on your daily
driver. openshell is the missing isolation layer:

- **Agent isolation** — run Claude (`ANTHROPIC_API_KEY`), Codex (`OPENAI_API_KEY`),
  Copilot (`GITHUB_TOKEN`), or OpenCode in a throwaway container, creds
  auto-discovered from the shell env and held centrally as **Providers**.
- **Inference routing** — `openshell inference set --provider <p> --model <m>`
  intercepts the sandbox's model calls, strips the caller's credentials, and injects
  the managed backend's — so you can point every sandboxed agent at a local
  [[Local LLM Inference|inference endpoint]] (e.g. Ollama) or a controlled API, with
  hot-reload and no restart.
- **No key leakage** — the agent can run, but it can't exfiltrate your API keys or
  reach arbitrary hosts.

## Architecture

| Layer | What it is |
|-------|------------|
| **Gateway (control plane)** | `openshell-gateway` — native host process (not a container). Auth boundary for the CLI, sandbox lifecycle, TLS/JWT, SQLite state (`gateway.db`). |
| **Compute driver** | `docker` / `podman` / `kubernetes` / `vm`. Auto-detected (k8s → podman → docker → vm); launches sandboxes on demand. |
| **Sandbox** | One container per agent, supervised by `openshell-sandbox`, with policy-enforced egress. |

Four **policy layers** govern a sandbox — **filesystem**, **network**, **process**,
and **inference**. Filesystem/process are locked at creation; network/inference are
**hot-reloadable** (`openshell policy set <name> --policy file.yaml --wait`).

Built in **Rust** (gateway + supervisor; uses the Z3 solver via an `openshell-prover`
crate for policy verification), with a `mise`-managed toolchain.

## Day-to-day

```sh
mise run gateway                      # build + run the dev gateway (127.0.0.1:18080)
openshell gateway select docker-dev   # point the CLI at it
openshell status

openshell sandbox create -- claude    # launch Claude in a sandbox
openshell sandbox create --from ollama# from the community catalog
openshell sandbox list
openshell sandbox connect <name>      # shell into a running sandbox
openshell logs <name> --tail

openshell provider create --type <t> --from-existing   # register creds
openshell inference set --provider <p> --model <m>      # route model calls
openshell term                        # live TUI dashboard
```

> [!note]
> The official installer only handles `dpkg`/`rpm`, so on **Arch** you build from
> source (`pacman -S z3 mise`, then `git clone … && mise trust && mise install`).
> The dev gateway runs **unauthenticated on `127.0.0.1:18080`** — fine for local
> use, **not** a production posture. Telemetry is on by default
> (`OPENSHELL_TELEMETRY_ENABLED=false` to disable).

## How it relates to neighbours

- **vs [[Docker]] / [[NVIDIA Container Toolkit]]** — those *run* containers (and
  inject GPUs); openshell adds an **agent-aware policy/egress/credential layer** on
  top, with inference routing baked in.
- **vs [[hermes-agent]]'s own sandboxing** — Hermes has Docker/SSH terminal backends
  and approval gates; openshell is a dedicated, policy-first control plane you could
  host such agents inside for stronger network/credential guarantees.

## Pros / trade-offs

**Pros**
- Purpose-built isolation for autonomous agents (the thing you actually want).
- Credential stripping/injection + three-outcome egress = real zero-trust.
- Driver-agnostic (Docker/Podman/K8s/VM); hot-reloadable network/inference policy.

**Trade-offs**
- Young NVIDIA project; dev gateway is unauthenticated/plaintext (harden for real use).
- Source build on Arch (no native package); Z3 + mise toolchain dependency.
- Another control plane to operate alongside your container runtime.

## Related

- [[hermes-agent]] — the kind of autonomous agent openshell is built to contain
- [[Docker]] · [[NVIDIA Container Toolkit]] — the runtime layer it sits above
- [[Local LLM Inference]] — route sandboxed agents at a local model backend
- [[Zero Trust Networking]] — the egress/credential model in one phrase
