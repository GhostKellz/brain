---
type: entity
title: GhostCP
created: 2026-06-21
updated: 2026-06-28
tags:
  - rust
  - hosting
  - control-panel
  - dns
  - dnssec
status: experimental
entity_type: repository
language: Rust
repo_status: experimental
purpose: Host-level hosting control panel in Rust — HestiaCP reimagined with
  Axum, Leptos, and standards-based DNS interop
related:
  - "[[HestiaCP]]"
  - "[[Rust]]"
  - "[[Databases]]"
  - "[[Cloudflare]]"
  - "[[acme.sh - DNS-01 Certificates]]"
  - "[[CrowdSec]]"
  - "[[Wazuh]]"
  - "[[Tailscale]]"
  - "[[Nginx Reference]]"
  - "[[Self-Hosted Services]]"
  - "[[Ghost Ecosystem]]"
  - "[[GhostKellz]]"
---

# GhostCP

A host-level **hosting control panel** written in [[Rust]] — inspired by
[[HestiaCP]] but built around a memory-safe control plane and standards-based DNS
interoperability. Marked experimental (lab / non-critical hosts). By
[[GhostKellz]], MIT-licensed.

## Stack

- **API**: Axum 0.8 REST control plane.
- **UI**: Leptos 0.8 (SSR + WASM), Tera templates.
- **Database**: PostgreSQL 16 as system-of-record (see [[Databases]]), SQLx
  migrations.
- **Auth**: JWT + Argon2, TOTP 2FA with backup codes, RBAC.

## DNS subsystem (functional)

- Zone and record CRUD.
- Zone transfer (AXFR/IXFR), NOTIFY push, TSIG authentication.
- DNSSEC signing.
- Pluggable drivers: [[Cloudflare]] API, PowerDNS API, and local BIND.
- Mix secondaries freely with no lock-in.

## Web hosting (scaffolded)

- NGINX vhost templates (see [[Nginx Reference]]).
- Automatic TLS via ACME / Let's Encrypt — DNS-01 for wildcards (see
  [[acme.sh - DNS-01 Certificates]]) and HTTP-01.
- HTTP/2; WordPress with isolated PHP-FPM pools.

## Control plane

- REST API with RBAC, Prometheus metrics, and systemd-native service templating.
- Mail, database, and backup managers are scaffolded (routes + schema present,
  logic in progress).

## Planned

- [[CrowdSec]] IPS bouncers and [[Wazuh]] SIEM integration.
- Loki / Prometheus / Grafana log and metric shipping.
- [[Tailscale]]-private admin plane.

## Status

v0.1.0 — early and experimental. Control plane, auth, and DNS are functional; the
rest is scaffolded. For lab and non-critical hosts. MIT-licensed.

> [!key-insight]
> GhostCP favors standards-based DNS interoperability (AXFR/IXFR, TSIG, DNSSEC,
> pluggable drivers) over proprietary lock-in — a deliberate contrast to
> single-vendor panels.

## Relationships

- Inspired by [[HestiaCP]]; built on [[Rust]] with PostgreSQL [[Databases]].
- DNS drivers include [[Cloudflare]]; TLS via [[acme.sh - DNS-01 Certificates]].
- Planned security/observability via [[CrowdSec]], [[Wazuh]], and a
  [[Tailscale]] admin plane.
- Part of the [[Ghost Ecosystem]]; fits the [[Self-Hosted Services]] space.
