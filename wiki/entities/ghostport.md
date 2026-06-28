---
type: entity
title: "ghostport"
created: 2026-06-21
updated: 2026-06-21
tags:
  - rust
  - docker
  - registry
  - oci
status: seed
entity_type: repository
language: Rust
repo_status: experimental
purpose: "Modern Docker/OCI registry in Rust with an optional SvelteKit management UI"
related:
  - "[[Rust]]"
  - "[[Ghost Ecosystem]]"
  - "[[GhostCP]]"
  - "[[ghostcrate]]"
  - "[[Docker]]"
  - "[[GhostKellz]]"
---

# ghostport

A modern **Docker/OCI registry** written in [[Rust]], with an optional SvelteKit
management UI. The Rust server and the SvelteKit UI run as separate services. By
[[GhostKellz]].

## Registry features

- Full OCI Distribution Spec compliance
- Multi-architecture images (manifest lists, platform selection)
- Layer deduplication and efficient storage
- Garbage collection without downtime
- Tag immutability options
- Webhook support for CI/CD

## Security

- Vulnerability scanning via integrated Trivy (CVE detection)
- Signature verification via Cosign (image signing/verification)
- Configurable per-client rate limiting

## Stack

Rust server · SvelteKit + TypeScript UI · PostgreSQL · S3-compatible storage ·
Tailwind CSS.

Part of the [[Ghost Ecosystem]]; complements the control panel [[GhostCP]] and
the crate tooling [[ghostcrate]]. See [[Docker]] for related
container workflow.
