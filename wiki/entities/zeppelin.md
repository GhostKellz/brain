---
type: entity
title: zeppelin
created: 2026-06-28
updated: 2026-06-28
tags:
  - zig
  - package-manager
  - registry
  - self-hosted
status: developing
entity_type: repository
language: Zig
repo_status: beta
purpose: Lightweight package manager and self-hosted registry for the Zig ecosystem
related:
  - "[[Zig]]"
  - "[[zqlite]]"
  - "[[Self-Hosted Services]]"
  - "[[Nginx Reference]]"
  - "[[GhostKellz]]"
  - "[[CK Technology LLC]]"
---

# zeppelin

A lightweight package manager paired with a self-hosted registry for the Zig
ecosystem — a Cargo-for-Zig with a private registry option. Authored by
[[GhostKellz]] / [[CK Technology LLC]], MIT-licensed.

> [!key-insight]
> The repository directory is spelled `zepplin` and the project config file is
> `zepplin.toml`; the canonical project name used here is "zeppelin".

## Stack

- **Language**: Zig (0.17.0-dev), ~9.6k lines.
- **Storage**: depends on [[zqlite]] v1.7.0.
- **Config**: TOML (`zepplin.toml`) plus a lockfile.
- **Web UI**: dark-themed registry frontend.
- **Deployment**: Docker multi-stage build with an nginx reverse proxy;
  Docker Compose dev/prod profiles.

## Capabilities

- CLI: `init`, `add`, `update`, `build`, `publish`, `login`, `serve`, `browse`,
  `trending`.
- Self-hosted registry server with a REST API.
- Web UI with real-time search, usage stats, and a package browser.
- Multiple auth backends: local, OIDC, Entra, GitHub, Google.
- SQLite plus in-memory DB backends with fallback.
- Zigistry integration for discovery; zigllibs importer.

## Status

v0.6.x working prototype with all three components (CLI, registry, web UI)
implemented; recently migrated to Zig 0.17. MIT. Roadmap: dependency resolution
with `zig build` integration, analytics, and vulnerability scanning.

## Relationships

Built on [[Zig]] and backed by [[zqlite]]. Runs as one of the
[[Self-Hosted Services]], typically fronted by an nginx reverse proxy (see
[[Nginx Reference]]). Shares the Zig packaging theme with [[zaur]].
