# 👻 GhostKellz Brain

<p align="center">
  <strong>An AI-queryable, cross-referenced second brain — markdown in git, readable by humans and by Claude / Codex.</strong>
</p>

<p align="center">
  <em>Knowledge & AI stack</em><br>
  <img src="https://img.shields.io/badge/Obsidian-7C3AED?style=for-the-badge&logo=obsidian&logoColor=white" alt="Obsidian">
  <img src="https://img.shields.io/badge/Markdown-000000?style=for-the-badge&logo=markdown&logoColor=white" alt="Markdown">
  <img src="https://img.shields.io/badge/Claude-D97757?style=for-the-badge&logo=anthropic&logoColor=white" alt="Claude">
  <img src="https://img.shields.io/badge/Codex-412991?style=for-the-badge&logo=openai&logoColor=white" alt="Codex">
  <img src="https://img.shields.io/badge/Ollama-000000?style=for-the-badge&logo=ollama&logoColor=white" alt="Ollama">
  <img src="https://img.shields.io/badge/MCP-Filesystem-1B5E20?style=for-the-badge&logo=modelcontextprotocol&logoColor=white" alt="MCP">
</p>

<p align="center">
  <em>Infrastructure</em><br>
  <img src="https://img.shields.io/badge/Arch_Linux-1793D1?style=for-the-badge&logo=arch-linux&logoColor=white" alt="Arch Linux">
  <img src="https://img.shields.io/badge/Proxmox-E57000?style=for-the-badge&logo=proxmox&logoColor=white" alt="Proxmox">
  <img src="https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white" alt="Docker">
  <img src="https://img.shields.io/badge/Tailscale-242424?style=for-the-badge&logo=tailscale&logoColor=white" alt="Tailscale">
  <img src="https://img.shields.io/badge/NVIDIA-RTX_5090-76B900?style=for-the-badge&logo=nvidia&logoColor=white" alt="NVIDIA RTX 5090">
</p>

<p align="center">
  <em>Project</em><br>
  <img src="https://img.shields.io/badge/Quartz-planned-8A2BE2?style=for-the-badge" alt="Quartz (planned)">
  <img src="https://img.shields.io/badge/GPG_Signed-Verified-00C853?style=for-the-badge&logo=gnu-privacy-guard&logoColor=white" alt="GPG Signed">
  <img src="https://img.shields.io/badge/License-MIT-A31F34?style=for-the-badge" alt="MIT License">
  <img src="https://img.shields.io/badge/GhostKellz-👻-9B59B6?style=for-the-badge" alt="GhostKellz">
</p>

---

> 🧠 **About this repository:**
> A public, cross-referenced knowledge layer over my systems, dev ecosystem, MSP
> operations, and cross-platform notes (Linux / macOS / Windows). It mirrors the
> [`arch`](https://github.com/ghostkellz/arch) pattern — a markdown knowledge base
> owned outright, queryable by [Claude](https://claude.com/claude-code) and Codex
> through a filesystem MCP, with retrieval embeddings generated locally.

> 📓 **Note:** Treat this as a living **knowledge base**, not authoritative docs.
> Details (versions, commands, topology labels) reflect a point in time and drift.

---

## ⚠️ This repo is PUBLIC

Only public-safe knowledge is *committed* here. The vault on disk may also hold
private notes, but they live in gitignored paths and never leave the machine. The
tiering is strict and load-bearing — because the public tier is meant to ship as a
website, anything committed here is effectively published.

| Tier | What | Lives in |
|------|------|----------|
| 🟢 **Public** (tracked) | general cheatsheets, cross-platform notes, OSS project docs | GitHub `ghostkellz/brain` |
| 🟡 **Personal / home-lab / security** | heimdall-stack, security configs, infra inventory | gitignored `private/` paths in this vault — local-only, never pushed |
| 🔴 **Client-confidential** | per-client runbooks, topologies, M365 tenants | **Hudu** (`hudu.cktechx.com`) — system of record, never in git |

Never commit home-lab topology, security configs, secrets, or client data here.
When in doubt, it goes in a gitignored `private/` path or Hudu — not the tracked
public tier.

---

## 📊 What's inside

8 domains · ~150 cross-linked pages.

| Type | Count | What |
|------|-------|------|
| 🏛️ **Domains** | 8 | top-level topic hubs |
| 🔖 **Entities** | 52 | people, orgs, products, repos, tools |
| 💡 **Concepts** | 39 | ideas, patterns, frameworks |
| 📒 **References** | 39 | runbooks and how-tos |
| 📥 **Sources** | 3 | one summary per raw source |

**Domains:** Linux & Systems · AI & Local LLMs · Networking · Cloud & Microsoft 365 ·
Security · Web Development · Programming Languages · DevOps & Homelab.

---

## 🏗️ Architecture

The brain is a *derived* layer. Sources of truth stay canonical; the brain ingests
and cross-references them.

```
Sources of truth                 Derived AI layer
────────────────                 ────────────────
~/arch          (system/lab) ──► ~/brain ──► GitHub (public tier only)
/data/projects  (code)       ──► (this)      private/ paths stay local (gitignored)
Hudu            (clients)    ──►              MemPalace (agent graph memory)
```

### Knowledge stack (under evaluation)

- **claude-obsidian** — primary knowledge wiki: markdown, filesystem MCP transport,
  local Ollama embeddings via `nomic-embed-text`. Human-browsable, cross-linked.
- **MemPalace** — agent graph-memory contender, kept running for a head-to-head eval.
- **second-brain-starter** (coleam00) — later candidate for proactive agentic
  features (heartbeat, notifications) layered on top of this vault.

---

## 📂 Layout

```
brain/
├── .raw/             # immutable source inputs awaiting ingestion
├── wiki/             # the knowledge base (LLM-generated, human-edited)
│   ├── index.md      # master catalog
│   ├── overview.md   # executive summary
│   ├── hot.md        # recent-context cache (~500 words)
│   ├── log.md        # append-only operations log
│   ├── entities/     # people, orgs, products, repos
│   ├── concepts/     # ideas, patterns, frameworks
│   ├── references/   # runbooks & how-tos
│   ├── sources/      # one summary page per raw source
│   ├── domains/      # top-level topic hubs
│   ├── comparisons/  # side-by-side analyses
│   ├── questions/    # filed answers to queries
│   └── meta/         # dashboards, lint reports
├── _templates/       # note templates
├── WIKI.md           # the schema (Layer 3)
└── tasks/            # local-only planning (gitignored)
```

Every page carries YAML frontmatter; links are `[[Wikilinks]]`, not paths; one
concept per page. Start at [`wiki/hot.md`](wiki/hot.md) for recent context, then
[`wiki/index.md`](wiki/index.md) for the full catalog.

---

## 🚀 Publishing → brain.ckelley.dev

The public tier is intended to ship as a self-hosted, browsable knowledge site at
`brain.ckelley.dev`. Likely path: render the vault to a static site preserving
wiki-links + graph view (**Quartz**, Obsidian-native; or Astro to match
`ckelley.dev` / `ghostkellz.sh`), deployed via CI on push to `main`. Only the
public tier ever renders — the tiering rule above guarantees no leakage.

---

## 🔍 Retrieval

Local-first. Embeddings run on the RTX 5090 via Ollama (`nomic-embed-text`); no
page bodies leave the machine. Contextual retrieval to the Anthropic API stays
**off** by default and is only ever enabled for content in this public tier.

---

### 🔐 GPG Commit Signing

This repository uses verified commits with a WKD-compliant key.

**Author GPG Key:** `ckelley@ghostkellz.sh`

```bash
# Import via WKD
gpg --locate-keys ckelley@ghostkellz.sh
```

The public key can be viewed and downloaded at **[ghostkellz.sh](https://ghostkellz.sh)**.

---

## 📄 License

[MIT](LICENSE) © 2026 CK Technology LLC.

---

### 🔍 Maintained by [Christopher Kelley](https://github.com/ghostkellz) · [CK Technology LLC](https://cktechx.com)

<p align="center"><b>Local-first infrastructure · Rust for apps · Zig for engines.</b></p>
