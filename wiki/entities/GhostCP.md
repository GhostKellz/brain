---
type: entity
title: "GhostCP"
created: 2026-06-21
updated: 2026-06-21
tags:
  - rust
  - hosting
  - control-panel
  - dns
status: seed
entity_type: repository
language: Rust
repo_status: experimental
purpose: "Host-level hosting control panel in Rust — HestiaCP reimagined with Axum, Leptos, and standards-based DNS"
related:
  - "[[Rust]]"
  - "[[Ghost Ecosystem]]"
  - "[[ghostport]]"
  - "[[Self-Hosted Services]]"
  - "[[HestiaCP]]"
  - "[[GhostKellz]]"
---

# GhostCP

A host-level **hosting control panel** written in [[Rust]] — [[HestiaCP]]
reimagined with a memory-safe control plane. Marked experimental (research / lab
use). By [[GhostKellz]].

## What it does

Manages a single server's web, DNS, mail, database, and TLS configuration the way
HestiaCP does — by templating native services (NGINX, PHP-FPM, BIND/PowerDNS,
Postfix/Dovecot) and driving them with systemd — but with a Rust control plane, a
PostgreSQL system-of-record, and a Leptos web UI.

## Stack

- **Control plane**: Rust + Axum 0.8
- **UI**: Leptos 0.8
- **Database**: PostgreSQL 16
- **Web**: NGINX (templated)
- **DNS**: BIND / PowerDNS
- **TLS**: Let's Encrypt / ACME
- **Security**: CrowdSec, Wazuh, threat feeds

Favors standards-based interoperability over proprietary lock-in. Part of the
[[Ghost Ecosystem]]; complements the registry [[ghostport]]. See
[[Self-Hosted Services]] for the wider self-hosting context.
