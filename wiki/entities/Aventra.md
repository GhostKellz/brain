---
type: entity
title: "Aventra"
created: 2026-06-28
updated: 2026-06-28
tags:
  - crm
  - marketing-automation
  - ai
  - go
  - saas
status: developing
entity_type: repository
language: Go
repo_status: experimental
purpose: "Self-hostable, AI-native growth-operations platform unifying CRM, lead intelligence, AI agents, marketing automation, and analytics for service businesses."
related:
  - "[[Go]]"
  - "[[Databases]]"
  - "[[Docker]]"
  - "[[GhostKellz]]"
---

# Aventra

A self-hostable, AI-native growth-operations platform that unifies CRM, lead
intelligence, AI agents, marketing automation, and analytics for service
businesses. By [[GhostKellz]].

> [!key-insight]
> Aventra is an explicit proof of concept — single-tenant by design, with a
> durable outbox queue and a manual approval gate so no outreach leaves the
> system without a human in the loop.

## Stack

- [[Go]] (chi router), Go 1.26; two binaries (API + worker)
- Frontend: Astro / Alpine.js / TypeScript
- Data: PostgreSQL (pgx, sqlc), Redis ([[Databases]]); goose migrations

## Capabilities

- CRM core: companies, contacts, leads, and a polymorphic activity timeline
- Email / outreach over SMTP with credentials encrypted AES-256-GCM at rest
- Manual approval gate before any send
- Durable outbox queue drained by a dedicated worker
- Local Argon2id authentication plus OIDC (Entra / Google)
- REST API with consistent JSON error envelopes

## Status

- Proof of concept — explicitly not production ready
- Breaking changes expected; single-tenant by design
- FSL-1.1-ALv2 license (converts to Apache-2.0 after 2 years)

## Relationships

- Built on [[Go]] with PostgreSQL / Redis ([[Databases]])
- Containerized with [[Docker]]
