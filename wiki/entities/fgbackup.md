---
type: entity
title: "fgbackup"
created: 2026-06-28
updated: 2026-06-28
tags:
  - fortigate
  - backup
  - forticloud
  - python
  - restic
status: developing
entity_type: repository
language: Python
repo_status: experimental
purpose: "Automated FortiGate firewall configuration backups pulled from the FortiCloud API, then encrypted and versioned with restic to S3-compatible object storage."
related:
  - "[[FortiGate Administration]]"
  - "[[Python]]"
  - "[[Restic Backup]]"
  - "[[3-2-1 Backup Strategy]]"
  - "[[GhostKellz]]"
  - "[[CK Technology LLC]]"
---

# fgbackup

Automated FortiGate firewall configuration backups: pull configs from the
FortiCloud API, then encrypt and version them with restic into S3-compatible
object storage. By [[GhostKellz]] / [[CK Technology LLC]].

> [!key-insight]
> fgbackup isolates partial failures — one unreachable device does not abort the
> run — and leans on restic for encrypted, deduplicated, versioned snapshots that
> slot into a [[3-2-1 Backup Strategy]].

## Stack

- [[Python]] 3.13; httpx; pydantic / pydantic-settings
- restic (encrypted, deduplicated, versioned) — see [[Restic Backup]]
- supercronic scheduler
- Docker / Compose; pytest + ruff

## Capabilities

- FortiCloud API client (OAuth password grant)
- Enumerate all FortiGates in an account; per-device config pull
- Auto gzip decompression (FortiOS 7.0+)
- Resilience: proactive token refresh, exponential backoff with jitter on 429 / 5xx, honors Retry-After
- Partial-failure isolation; pre-backup validation
- restic-managed retention (e.g. 14 daily / 8 weekly / 12 monthly)
- CLI: poll / backup / forget / prune / check / run
- JSON-line structured logging

## Status

- v0.1.0, early development
- Poller, restic store, and containerized scheduler implemented and tested
- MIT licensed

## Relationships

- Backs up [[FortiGate Administration]] configurations
- Written in [[Python]]
- Versioning via [[Restic Backup]], fitting a [[3-2-1 Backup Strategy]]
