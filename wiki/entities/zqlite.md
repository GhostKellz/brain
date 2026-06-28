---
type: entity
title: zqlite
created: 2026-06-21
updated: 2026-06-28
tags:
  - database
  - zig
  - embedded
  - sql
status: developing
entity_type: repository
language: Zig
repo_status: active
purpose: Embedded SQL database written in pure Zig with SQLite-style simplicity
  and some PostgreSQL conveniences
related:
  - "[[Zig]]"
  - "[[zcrypto]]"
  - "[[Databases]]"
  - "[[Jarvis]]"
  - "[[Ghost Ecosystem]]"
  - "[[GhostKellz]]"
---

# zqlite

An embedded SQL database engine written in pure [[Zig]]. It aims for
SQLite-style simplicity (single-file / in-memory, no server) while pulling in a
few PostgreSQL conveniences. Authored by [[GhostKellz]] (CK Technology LLC),
MIT-licensed. Current version is **1.7.0 (beta)**. See [[Databases]] for related
storage notes.

> [!key-insight]
> The headline design constraint is **zero external Zig package dependencies** —
> the engine, parser, storage, runtime, and C ABI are all in-tree. As of v1.7.0
> it ships its **own async runtime**, replacing the former external `zsync`
> dependency, so the zero-deps property still holds.

## Language & stack

- **Language**: Zig (minimum `0.17.0-dev`, pinned in `build.zig.zon`).
- **Distribution**: consumed as a Zig module via `zig fetch --save`, plus a
  stable C FFI surface for embedding from C and other languages.
- **Build profiles** (`-Dprofile=`): `core`, `advanced` (default), and
  `experimental` (full).
- **Granular flags**: `-Dcrypto`, `-Dliboqs`, `-Dtransport`, `-Djson`,
  `-Dperformance`, `-Dconcurrent`, and `-Dffi` for fine-grained builds.

## Architecture

Source is organized by subsystem under `src/`:

- `db`, `parser`, `executor` (including window functions)
- `storage` — B+tree with WAL, recovery, and integrity checks
- `indexing`, `concurrent` (MVCC-style isolation)
- `json`, `sqlite_compat`
- `performance` (query cache + connection pool)
- `shell` (REPL), `ffi`, `logging`, `error_handling`
- `runtime` — the in-tree async runtime (replaced external `zsync`)
- `crypto`, `transport`
- `cluster`, `distributed`, `wallet` — experimental scaffolding

## Stable features

- Full CRUD; JOINs; GROUP BY / ORDER BY / LIMIT / OFFSET
- Aggregates including STDDEV / VARIANCE / GROUP_CONCAT
- Window functions (PARTITION BY, frames); subqueries
- UNION / INTERSECT / EXCEPT; DISTINCT; HAVING; RETURNING
- ON CONFLICT / UPSERT; SAVEPOINT / RELEASE / ROLLBACK TO
- Named params (`:name` / `@name` / `$name`); foreign keys
- CHECK / UNIQUE constraints; generated columns; defaults
- Partial and expression indexes
- B+tree + WAL with recovery and integrity checks
- ATTACH / DETACH with schema-qualified access
- PRAGMAs (`user_version`, `schema_version`, `table_info`, `integrity_check`)
- VACUUM and `vacuumInto`; read-only / immutable opens; busy timeout
- Cursor API; resource limits and progress callbacks
- Prepared-statement schema-version expiration
- FTS5-style virtual tables (MATCH); query cache; connection pool
- ANALYZE and EXPLAIN QUERY PLAN
- Stable C ABI (v1.0) with a symbol manifest

## Experimental features (opt-in, not stable)

Explicitly flagged as not stable in the README:

- Post-quantum crypto (ML-KEM-768 / ML-DSA-65 via a stdlib backend, with KAT /
  NIST test vectors; a liboqs path exists as a placeholder and is not linked)
- Simulated PQ-QUIC transport
- Clustering / distributed scaffolding
- Encryption-at-rest building blocks (ChaCha20-Poly1305 / field-level)

> [!gap]
> The crypto and transport modules ship building blocks, but transparent
> database-at-rest encryption is still **not** a stable public feature in
> v1.7.0. Treat at-rest encryption as not-yet-stable when citing this repo.

## Quality / testing signals

- Many `zig build test-*` targets: comprehensive, security, storage,
  transaction, sql-conformance, sqlite-diff, pqc, c-api, memory-leaks, window,
  and advanced.
- Parser fuzzer (50k iterations, zero leaks).
- Benchmarks with regression gates.
- Release validation (`zig build check`, SBOM, checksums, signatures).
- Docker-based test images.

## Status

Beta (1.7.0). Core SQL, window functions, transactions, indexing, the
performance layer, and the C ABI are stable and tested; crypto, transport, and
clustering remain experimental. Under active development.

## Relationships

- Built on [[Zig]]; embeds cleanly via its stable C ABI.
- Shares the cryptography theme with [[zcrypto]] for post-quantum primitives.
- A natural local-storage backend for memory/agent tooling such as [[Jarvis]];
  part of the broader [[Ghost Ecosystem]] storage story. See [[Databases]].
