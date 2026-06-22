# brain

GhostKellz second brain — an AI-queryable, cross-referenced knowledge layer over my
systems, dev work, and cross-platform notes (Linux / macOS / Windows).

This mirrors the [`arch`](https://github.com/ghostkellz/arch) pattern: a markdown
knowledge base in git, owned outright, readable by humans and by Claude / Codex.

## ⚠️ This repo is PUBLIC

Only public-safe knowledge lives here. The tiering is strict:

| Tier | Lives in | Host |
|------|----------|------|
| **Public** (this repo) | general cheatsheets, cross-platform notes, dev project docs | GitHub `ghostkellz/brain` |
| **Personal / home-lab / security** | heimdall-stack, security configs, infra inventory | `~/brain-private` → GitLab `git.cktechx.com` (private) |
| **Client-confidential** | per-client runbooks, topologies, M365 tenants | **Hudu** (`hudu.cktechx.com`) — system of record, never in git |

Never commit home-lab topology, security configs, secrets, or client data here.
When in doubt, it goes in the private vault or Hudu, not here.

## Architecture

The brain is a *derived* layer. Sources of truth stay canonical; the brain ingests
and cross-references them:

```
Sources of truth                 Derived AI layer
────────────────                 ────────────────
~/arch        (system/lab)  ──►  ~/brain   ──► GitHub (public)
/data/projects (code)       ──►  (this)         ~/brain-private ──► GitLab (private)
Hudu          (clients)     ──►                 MemPalace (agent graph memory)
```

### Knowledge stack (under evaluation)

- **claude-obsidian** — primary knowledge wiki (markdown, MCP filesystem transport,
  local ollama embeddings via `nomic-embed-text`). Human-browsable, cross-linked.
- **MemPalace** — agent graph memory contender, kept running for head-to-head eval.
- **second-brain-starter** (coleam00) — Phase 2 candidate for proactive agentic
  features (heartbeat, notifications) layered on top of this vault.

## Layout

```
cheatsheets/   cross-platform notes (Linux / macOS / Windows)
dev/           dev project docs (zqlite, ghostkellz.sh, ckelley.dev, ...)
sources/.raw/  raw inputs awaiting ingestion
tasks/         planning + lessons (todo.md is the build spec)
```

### Topic areas

DevOps · AI · Coding · Web Dev · App Dev · Networking (Tailscale, FortiGate) ·
Cloud (Azure) · Security. These shape the public knowledge taxonomy.

## Publishing → brain.ckelley.dev

The public tier is intended to ship as a self-hosted, browsable knowledge site at
`brain.ckelley.dev` — a differentiator others can leverage. Likely path: render the
vault as a static site (Quartz, or Astro to match `ckelley.dev` / `ghostkellz.sh`).
Because publishing turns *everything in this repo* into a public website, the tiering
rule above is load-bearing: only public-safe content ever enters this vault.

## Retrieval

Local-first. Embeddings run on the 5090 via ollama (`nomic-embed-text`); no page
bodies leave the machine. Contextual retrieval to the Anthropic API stays **off** by
default and is only ever enabled for content in this public tier.
