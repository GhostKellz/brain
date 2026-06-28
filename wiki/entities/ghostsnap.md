---
type: entity
title: ghostsnap
created: 2026-06-21
updated: 2026-06-28
tags:
  - backup
  - cli
  - linux
  - rust
  - encryption
status: active
entity_type: repository
language: Rust
repo_status: active
purpose: Fast, secure, deduplicating backup CLI for Linux with end-to-end
  encryption and multiple backends
related:
  - "[[Restic Backup]]"
  - "[[3-2-1 Backup Strategy]]"
  - "[[Rust]]"
  - "[[Self-Hosted Services]]"
  - "[[Ghost Ecosystem]]"
  - "[[GhostKellz]]"
---

# ghostsnap

A fast, secure, deduplicating backup CLI for Linux with a self-hosted focus.
Inspired by [[Restic Backup]] and built in [[Rust]] (2024 edition) as a workspace
of `cli`, `core`, and `backends` crates. By [[GhostKellz]], MIT-licensed.

## Language & stack

- **Language**: [[Rust]] (2024 edition), workspace layout (cli / core /
  backends).
- **Crypto**: ChaCha20-Poly1305 (AEAD) with an Argon2id KDF.
- **Dedup**: content-defined chunking via FastCDC with BLAKE3 content
  addressing.

## Backends

- Local filesystem.
- S3-compatible: AWS, Wasabi, Backblaze B2, MinIO.
- Azure Blob.
- Native SFTP (Ed25519/ECDSA with `known_hosts` verification).
- Rclone bridge (40+ providers).

## Capabilities

- **init** — create a repository.
- **backup** — tags, incremental snapshots, and full Unix metadata (permissions,
  ownership, hardlinks, symlinks, xattrs, sparse files) with progress and ETA.
- **snapshots** — list, `ls`, `diff`, and `dump`.
- **check** — BLAKE3 integrity verification.
- **stats** — repository statistics.
- **forget / prune** — retention by daily/weekly/monthly policy.
- **copy** — replicate snapshots across repositories.
- **jobs** — TOML job definitions with pre/post hooks.
- **restore** — selective restore of files and trees.

## Status

v0.1 — core complete and tested; single-writer today, with remote locking in
progress. MIT-licensed.

> [!key-insight]
> ghostsnap brings restic-style deduplication and AEAD encryption to a native
> [[Rust]] workspace, fitting a [[3-2-1 Backup Strategy]] across local, object,
> and SFTP targets.

## Relationships

- Inspired by [[Restic Backup]]; supports a [[3-2-1 Backup Strategy]].
- Built on [[Rust]]; part of the [[Ghost Ecosystem]].
- Fits the [[Self-Hosted Services]] data-protection space.
