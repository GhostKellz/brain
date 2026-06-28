---
type: concept
title: "Databases"
created: 2026-06-28
updated: 2026-06-28
tags:
  - databases
  - postgres
  - sql
  - redis
  - architecture
status: developing
related:
  - "[[Go]]"
  - "[[Python]]"
  - "[[Rust]]"
  - "[[Docker]]"
---

# Databases

Picking the right datastore is an architecture decision you live with for years.
This is the map: the major families, the specific engines worth knowing, and the
trade-offs that decide which to reach for when starting a new project.

> [!key-insight]
> Default to **PostgreSQL** unless you have a concrete reason not to. It's a
> relational database that also does JSON, full-text search, geospatial, queues,
> and more — so "boring Postgres" covers a huge fraction of projects before you
> need anything specialised. Add a second store (cache, queue, ledger) only when
> a real requirement forces it.

## The decision axes

- **Relational (SQL) vs non-relational (NoSQL)** — structured, related data with
  joins and constraints, vs flexible/denormalised documents or key-values.
- **OLTP vs OLAP** — many small transactions (app backends) vs large analytical
  scans (warehouses/reporting). Most engines here are OLTP.
- **ACID vs BASE** — strong transactional guarantees vs eventual consistency for
  scale/availability.
- **Embedded vs client/server** — runs in-process (SQLite) vs a separate daemon
  you connect to over the network (Postgres, Redis).
- **Durability vs speed** — disk-backed and crash-safe vs in-memory and fast.

## Relational engines

### PostgreSQL
The default open-source RDBMS: rock-solid ACID, the richest feature set, and a
huge extension ecosystem.

- **Strengths**: standards-compliant SQL, `JSONB` (document workloads without a
  second DB), full-text search, `PostGIS` (geo), window functions, CTEs, MVCC
  concurrency, logical replication, extensions (`pg_stat_statements`, `pgvector`
  for embeddings/RAG, `TimescaleDB` for time-series, `Citus` for sharding).
- **Trade-offs**: connection-per-backend model needs a pooler (`PgBouncer`) at
  scale; tuning (`shared_buffers`, `work_mem`, autovacuum) matters under load.
- **Reach for it when**: almost always — the safe default for a new app.

### MariaDB / MySQL
The other ubiquitous RDBMS; MariaDB is the community fork of MySQL.

- **Strengths**: extremely widespread, simple replication, fast for
  read-heavy/simple-query web workloads, great hosting/tooling support (the "M"
  in LAMP). MariaDB adds engines (Aria, ColumnStore) and stays GPL.
- **Trade-offs**: historically looser SQL semantics and fewer advanced features
  than Postgres (JSON, window functions came later); MySQL/MariaDB are diverging.
- **Reach for it when**: existing LAMP stacks, WordPress/CMS, or shops already
  standardised on it.

### SQLite
A serverless, single-file, embedded SQL database — the most-deployed database in
the world (every phone, browser, app).

- **Strengths**: zero-config, no daemon, the whole DB is one file; rock-solid,
  fast for local/embedded use; great for tests, CLIs, desktop/mobile apps, and
  small sites. `WAL` mode improves concurrent reads.
- **Trade-offs**: single-writer (writes serialise); not built for many concurrent
  network clients. Server-grade concurrency means moving to Postgres.
- **Reach for it when**: embedded/local storage, app config, on-device data,
  tests, or low-traffic single-node services. See also [[zqlite]] (the Zig SQLite).

## Key-value / cache / in-memory

### Redis
In-memory data-structure store used as cache, session store, queue, rate limiter,
and pub/sub.

- **Strengths**: sub-millisecond latency; rich types (strings, hashes, lists,
  sets, sorted sets, streams, bitmaps, HyperLogLog); TTL/eviction; pub/sub and
  Streams for messaging; Lua scripting. Optional persistence (RDB snapshots / AOF).
- **Trade-offs**: dataset must largely fit in RAM; durability is weaker than a
  disk-first DB; it's a *complement* to your primary store, not a replacement.
  (Note the licensing shift to RSALv2/SSPL; **Valkey** is the open fork.)
- **Reach for it when**: caching hot data, sessions, job queues, leaderboards,
  rate limiting, ephemeral fast state.

## Specialised / newer engines

### TigerBeetle
A purpose-built, high-performance **financial transactions / accounting**
database (double-entry, debit/credit), written in [[Zig]].

- **Strengths**: extreme throughput for transfers; strict serializability;
  built-in double-entry accounting primitives; deterministic, fault-tolerant
  (cluster consensus). Solves the "money correctness at scale" problem that a
  general RDBMS does slowly and awkwardly.
- **Trade-offs**: single-purpose (ledgers/balances) — not a general datastore;
  you pair it with Postgres for everything that isn't accounting.
- **Reach for it when**: ledgers, payments, billing, in-app credits, anything
  that's fundamentally debit/credit at high volume.

### Turso / libSQL
**libSQL** is an open fork of SQLite that adds a server, replication, and remote
access; **Turso** is the hosted, edge-distributed offering on top.

- **Strengths**: SQLite's simplicity with network access, replicas, and
  edge/local-first sync; embedded replicas put a read copy next to the app for
  low-latency reads. Good fit for serverless/edge and multi-tenant (DB-per-tenant).
- **Trade-offs**: younger ecosystem; inherits SQLite's single-writer model
  per database; you're adopting a newer vendor/runtime.
- **Reach for it when**: edge/serverless apps, local-first sync, or you love
  SQLite but need it networked and replicated.

## Quick chooser

| Need | Pick |
|------|------|
| General app backend | **PostgreSQL** |
| Existing LAMP / WordPress | **MariaDB / MySQL** |
| Embedded / local / on-device / tests | **SQLite** (or [[zqlite]]) |
| Cache / sessions / queue / rate limit | **Redis / Valkey** |
| Ledgers / payments / accounting at scale | **TigerBeetle** (+ Postgres) |
| Edge / serverless / local-first SQLite | **Turso / libSQL** |
| Vector search / RAG embeddings | **Postgres + `pgvector`** |
| Time-series / metrics | **Postgres + TimescaleDB** (or Prometheus TSDB) |

## Design principles when architecting

- **One source of truth.** Keep the canonical data in one transactional DB
  (usually Postgres); treat caches/search indexes as derived and rebuildable.
- **Normalise first, denormalise for proven hot paths.** Don't pre-optimise the
  schema for scale you don't have.
- **Index deliberately.** Index what you filter/join/sort on; every index costs
  write throughput and storage.
- **Migrations are code.** Version schema changes (e.g. Alembic for [[Python]],
  sqlx/diesel for [[Rust]], golang-migrate for [[Go]]); never hand-edit prod.
- **Pool connections.** App frameworks + `PgBouncer` for Postgres; respect the
  engine's concurrency model.
- **Back it up and test restores.** A backup you haven't restored isn't a backup
  → [[3-2-1 Backup Strategy]], [[Restic Backup]]. Use native dumps (`pg_dump`,
  `mysqldump`) plus snapshots.
- **Run it right in containers.** Persist data on a named volume, not the
  container layer → [[Docker]].

## Related

- [[Docker]] — running DBs in containers (volumes, backups)
- [[3-2-1 Backup Strategy]] · [[Restic Backup]] — protecting the data
- [[zqlite]] — the Zig-built SQLite in this ecosystem
