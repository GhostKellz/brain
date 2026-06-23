---
type: reference
title: "Hudu"
created: 2026-06-23
updated: 2026-06-23
tags:
  - hudu
  - mcp
  - ai
  - documentation
  - msp
  - knowledge-base
status: developing
related:
  - "[[Ollama Service Configuration]]"
  - "[[Self-Hosted Services]]"
---

> [!key-insight] [Hudu](https://www.hudu.com/) is an MSP documentation platform
> (the system of record for client passwords, asset configs, and runbooks). Its
> **native MCP server** lets an AI coding agent — Claude Code, Codex — read and
> **write** that documentation directly, so you pull live client context and push
> doc updates from the same place you do the work, without copy-paste. All
> hostnames below are placeholders (`hudu.example.com`); swap in your tenant.

> [!warning] **Tier discipline is the whole point.** Client data stays in Hudu
> (private, audited). General/publishable knowledge stays in the public vault.
> When you generalize a Hudu-sourced fix into the vault, capture the **pattern
> only** — never client names, hostnames, IPs, or other identifiers. Decide a
> note's tier *before* writing it, never after.

## Two-tier knowledge model

Each tier is reached through its own MCP, split by sensitivity:

| Tier | Lives in | Audience | Access path |
|------|----------|----------|-------------|
| **Client data** — passwords, asset configs, per-client runbooks | **Hudu** | private, audited | Hudu MCP (OAuth, scoped) |
| **Public / general knowledge** — systems, dev, cross-platform notes | public vault → static site | publishable, git-tracked | vault MCP + retrieval |
| **Private home-lab / security** | local-only vault tier (gitignored) | local-only | vault MCP, never pushed |

The trust boundary is enforced by **Hudu's OAuth scopes and permissions**, not by
discipline alone — but discipline keeps the generalized notes clean.

## Native MCP server

- Shipped in Hudu **2.43.0+**: **Streamable HTTP** transport, **OAuth** auth.
  It replaces the older SSE server — repoint any previously configured clients.
- Per-user OAuth means **no API key in any config file**; tokens live in each
  CLI's own credential store.
- Get the exact endpoint from **Hudu → Admin → External Apps → Hudu MCP** and use
  what that page shows (examples here assume `https://hudu.example.com/mcp`).

## Claude Code

Add the server over HTTP transport, user-scoped (available in every project):

```bash
claude mcp add --transport http --scope user hudu https://hudu.example.com/mcp
```

Authenticate inside a session, then approve the OAuth scopes in the browser:

```
/mcp   → select "hudu" → complete OAuth flow → approve scopes
```

Manage it:

```bash
claude mcp list          # confirm hudu is connected
claude mcp get hudu      # inspect config
claude mcp remove hudu   # remove
```

## Codex

Append an HTTP MCP block to `~/.codex/config.toml` (alongside any existing stdio
servers). OAuth tokens are stored by Codex, **not** in this file — keep it
secret-free:

```toml
[mcp_servers.hudu]
url = "https://hudu.example.com/mcp"
```

Authenticate / revoke via OAuth:

```bash
codex mcp login hudu     # opens the browser OAuth flow
codex mcp logout hudu    # revoke
codex mcp list           # confirm hudu is registered
```

Optional per-server tuning if the endpoint is slow: `startup_timeout_sec`
(default 10), `tool_timeout_sec` (default 60). Builds before late-2025 needed
`experimental_use_rmcp_client = true` for native HTTP — current builds don't.

## Verify

- **Claude Code:** `claude mcp list` shows `hudu` connected; `/mcp` lists Hudu
  tools; a trivial read (e.g. list companies) returns.
- **Codex:** `codex mcp list` shows `hudu`; the same trivial read returns.
- **Scopes:** the OAuth approval screen granted the intended read/write level.

## Usage patterns

- **Pull client context live** instead of pasting it — look up a company's asset
  or runbook from Hudu while working a ticket.
- **Write back to the source of truth** — draft/update client docs, asset notes,
  and runbooks in Hudu (where they're versioned and audited), not in scratch.
- **Update internal KBs** — keep shared internal procedures current from the CLI.
- **Cross-tier work** — generalize a Hudu fix into a reusable pattern and file
  the *pattern* in the public vault (see the leak guard above); the client
  specifics stay in Hudu.

## Alternative — third-party API-key server (not preferred)

A self-hosted community server (stdio **or** HTTP) authed by a **Hudu API key**
(`HUDU_BASE_URL` + `HUDU_API_KEY` from **Admin → API Keys**) exposes more tools,
but the API key bypasses per-user OAuth scoping and lives in env/config. Prefer
the native OAuth server; reach for this only if the native tool set proves
insufficient.

## Related

- [[Self-Hosted Services]] — where a self-hosted docs/KB platform fits.
- [[Ollama Service Configuration]] — the local-LLM side of the AI tooling.
