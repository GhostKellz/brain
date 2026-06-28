---
type: entity
title: "Citadel"
created: 2026-06-28
updated: 2026-06-28
tags:
  - crowdsec
  - security
  - control-plane
  - go
  - observability
status: developing
entity_type: repository
language: Go
repo_status: active
purpose: "Performance-focused multi-site control plane for managing CrowdSec Security Engines and bouncers at scale."
related:
  - "[[CrowdSec]]"
  - "[[Go]]"
  - "[[Databases]]"
  - "[[Prometheus Monitoring]]"
  - "[[Grafana]]"
  - "[[GhostKellz]]"
---

# Citadel

A performance-focused, multi-site control plane for managing [[CrowdSec]]
Security Engines and bouncers at scale. By [[GhostKellz]].

> [!key-insight]
> Citadel ships as a single binary with the SvelteKit UI embedded via `go:embed`,
> and keeps dashboards fast through server-side pagination and pre-aggregated
> rollups rather than querying raw events on every load.

## Stack

- [[Go]] (Go 1.26) with an embedded SvelteKit frontend (`go:embed`)
- PostgreSQL ([[Databases]]); goose migrations
- [[Prometheus Monitoring]] metrics
- Single-binary distribution

## Capabilities

- Server-side pagination and pre-aggregated rollups for dashboards
- Multi-tenant / multi-site hierarchy with scoped RBAC
- SSO (OIDC / OAuth2: Entra / Google / GitHub) plus a local break-glass admin
- Live updates via Server-Sent Events (SSE)
- Prometheus metrics exporter with provisioned [[Grafana]] dashboards
- Pluggable source interface ([[CrowdSec]] is the first implementation)

## Status

- Active development
- MIT licensed

## Relationships

- Manages [[CrowdSec]] engines and bouncers
- Built on [[Go]] with PostgreSQL ([[Databases]])
- Observability through [[Prometheus Monitoring]] and [[Grafana]]
