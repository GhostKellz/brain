---
type: entity
title: strix
created: 2026-06-28
updated: 2026-06-28
tags:
  - object-storage
  - s3
  - rust
  - minio-alternative
  - storage
status: developing
entity_type: repository
language: Rust
repo_status: active
purpose: S3-compatible object-storage server written in Rust with a modern WASM
  web console
related:
  - "[[Rust]]"
  - "[[Self-Hosted Services]]"
  - "[[Restic Backup]]"
  - "[[Kubernetes]]"
  - "[[GhostKellz]]"
  - "[[CK Technology LLC]]"
---

# strix

A self-hosted, S3-compatible object-storage server written in [[Rust]] with a
modern web console, positioned as a MinIO-style alternative. Authored by
[[GhostKellz]] / [[CK Technology LLC]], AGPL-3.0 licensed.

> [!key-insight]
> The whole server ships as a single ~15 MB binary written in 100% safe Rust
> (zero `unsafe`), exposing the S3 API on one port and the web console on
> another.

## Stack

- **Language**: Rust (2024 edition, 1.96+).
- **Console**: Leptos 0.7 compiled to WASM.
- **HTTP / admin API**: Axum.
- **S3 protocol**: `s3s`.
- **IAM store**: SQLite-backed.
- **Ports**: 9000 (S3 API) and 9001 (web console).

## Capabilities

- S3 API compatible with the AWS SDK, `s3cmd`, and `rclone`.
- Multipart uploads, pre-signed URLs, object versioning.
- Server-side encryption: SSE-S3 and SSE-C.
- IAM users/groups with JSON policies and bucket policies; AWS SigV4 auth;
  OIDC/SSO.
- Management surfaces: web console, `sx` CLI, and a REST admin API.
- Prometheus metrics and Kubernetes health checks.
- Audit logging, per-bucket CORS, rate limiting.
- v0.1.0 additions: Object Lock (WORM), lifecycle rules, event notifications via
  webhooks, SMTP alerts and usage reports.
- Planned: distributed mode with erasure coding, site replication, built-in TLS.

## Status

Proof-of-concept under active development, v0.1.0. AGPL-3.0.

## Relationships

Built on [[Rust]]. A natural storage target for [[Self-Hosted Services]] and a
candidate S3 backend for [[Restic Backup]]. Ships [[Kubernetes]] health checks
and Prometheus metrics for cluster deployment.
