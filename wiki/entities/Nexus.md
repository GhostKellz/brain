---
type: entity
title: "Nexus"
created: 2026-06-28
updated: 2026-06-28
tags:
  - ticketing
  - psa
  - msp
  - service-desk
  - go
  - svelte
status: developing
entity_type: repository
language: Go
repo_status: active
purpose: "Self-hosted service desk / ticketing and PSA system for IT providers and MSPs."
related:
  - "[[Go]]"
  - "[[Databases]]"
  - "[[Docker]]"
  - "[[Microsoft 365 Administration]]"
  - "[[GhostKellz]]"
  - "[[CK Technology LLC]]"
---

# Nexus

A full-featured, self-hosted service desk / ticketing and PSA system aimed at IT
providers and MSPs. It positions as a lighter alternative to Kaseya BMS / IT Flow
while keeping first-class reporting. By [[GhostKellz]] / [[CK Technology LLC]].

> [!key-insight]
> Nexus pairs a Go (Echo) API with a SvelteKit front end and leans on
> real-time WebSocket updates so ticket queues stay live without manual refresh.

## Stack

- Backend: [[Go]] (Echo framework), Go 1.25; RESTful API under `/api/v1/`
- Frontend: SvelteKit / TypeScript, Node 18+
- Data: PostgreSQL, Redis (see [[Databases]])
- Deploy: [[Docker]] Compose

## Capabilities

- Multi-tab ticket management with real-time WebSocket updates
- Email-to-ticket ingestion (M365 / Gmail — see [[Microsoft 365 Administration]])
- Time tracking with billable / non-billable distinction
- Client management and SLA tracking
- PDF / Excel reporting
- Workflow automation via triggers and actions
- SSO (Entra / Google / Okta) plus local authentication

## Status

- Production-leaning and actively developed, but not yet finished
- Ships CHANGELOG, CONTRIBUTING, and SECURITY documentation
- Proprietary license

## Relationships

- Built on [[Go]] with PostgreSQL / Redis ([[Databases]])
- Containerized with [[Docker]]
- Email integration ties into [[Microsoft 365 Administration]]
