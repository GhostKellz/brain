---
type: entity
title: "zqlite"
created: 2026-06-21
updated: 2026-06-21
tags:
  - database
  - zig
  - embedded
  - sql
status: seed
entity_type: repository
language: Zig
repo_status: active
purpose: "Embedded SQL database written in pure Zig with SQLite-style simplicity and some PostgreSQL conveniences"
related:
  - "[[Zig]]"
  - "[[zcrypto]]"
  - "[[Jarvis]]"
  - "[[Ghost Ecosystem]]"
  - "[[GhostKellz]]"
---

# zqlite

An embedded SQL database engine written in pure [[Zig]]. It aims for
SQLite-style simplicity (single-file / in-memory, no server) while pulling in a
few PostgreSQL conveniences. Authored by [[GhostKellz]] (CK Technology LLC),
MIT-licensed.

> [!key-insight]
> The headline design constraint is **zero external Zig package dependencies** —
> the engine, parser, storage, and C ABI are all in-tree.

## Language & stack

- **Language**: Zig (pins a specific `0.17.0-dev` minimum in `build.zig.zon`).
- **Distribution**: consumed as a Zig module via `zig fetch --save`, plus a C
  FFI surface (`include/`) for embedding from C and other languages.
- **Build profiles** (`-Dprofile=`):
  - `core` — minimal engine
  - `advanced` (default) — stable optional features + C ABI, no crypto/transport
  - `experimental` — explicitly experimental modules

## Architecture

Source is organized by subsystem under `src/`:

- `parser/` — SQL parser (SELECT, INSERT, UPDATE, DELETE, JOINs, aggregates,
  subqueries)
- `storage/` — B+ tree storage with write-ahead logging (WAL); file-based and
  in-memory (`:memory:`) databases
- `db/`, `indexing/`, `executor/` — query execution and indexing
- `sqlite_compat/` — SQLite compatibility layer
- `ffi/` — C FFI bindings
- `crypto/` — encryption building blocks (see Experimental below)
- `transport/`, `runtime/`, `concurrent/`, `distributed/`, `cluster/` —
  scaffolding for networked/clustered use cases
- `wallet/`, `zeppelin/`, `zsync/` — integration-oriented modules

## Stable features

- Prepared statements with parameter binding
- JOINs, GROUP BY, ORDER BY, LIMIT
- `RETURNING` clause and `ON CONFLICT` (upsert), including
  `DO UPDATE SET col = excluded.col`
- In-memory and file-based databases
- C FFI bindings

## Experimental features (opt-in, not stable)

- Post-quantum crypto scaffolding (ML-KEM, ML-DSA) — scaffolding only
- Transport-layer hooks
- Internal encryption primitives exist, but transparent database-at-rest
  encryption is **not** a stable public feature yet

> [!gap]
> The README markets "ChaCha20-Poly1305" and post-quantum crypto via badges, but
> the project text is explicit that these are experimental/scaffolding. Treat
> at-rest encryption as not-yet-stable when citing this repo.

## Status

Beta. Core CRUD, joins, aggregates, prepared statements, upserts, and the C ABI
are stable and tested; the project ships security tests, memory-leak detection
tests, and a fuzzer. Under active development (current line is the 1.6.x series).

## Quality / testing signals

- `zig build test`, `test-security`, `test-memory-leaks`, `test-comprehensive`
- Docker-based test images under `docker/`
- Fuzzer used to verify parser error-path memory safety
- Published `SECURITY.md` and a stability/compatibility policy covering API, ABI,
  and database-format boundaries

## Relationships

- Built on [[Zig]]; embeds cleanly via C ABI.
- Shares the cryptography theme with [[zcrypto]] (the ecosystem's Zig crypto
  library) for post-quantum primitives.
- A natural local-storage backend for memory/agent tooling such as [[Jarvis]]
  (which today uses SQLite + FTS5); part of the broader [[Ghost Ecosystem]]
  storage story.
