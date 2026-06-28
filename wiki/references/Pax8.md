---
type: reference
title: "Pax8"
created: 2026-06-28
updated: 2026-06-28
tags:
  - msp
  - cloud-marketplace
  - distribution
  - billing
  - api
  - microsoft-csp
status: developing
related:
  - "[[Microsoft 365 Services]]"
  - "[[Managed IT Services]]"
  - "[[Huntress]]"
  - "[[Avanan]]"
  - "[[CK Technology LLC]]"
---

> [!key-insight] Pax8 is the **distributor layer** for an MSP — one marketplace to
> quote, order, provision, and bill cloud products (Microsoft CSP, security,
> backup, etc.) with a **single monthly invoice** instead of a dozen vendor
> portals and bills. The value isn't any one product; it's collapsing the whole
> procure-to-bill supply chain into one pane of glass + an API.

**Pax8** is a cloud marketplace / distributor for managed service providers. MSPs
buy vendor subscriptions (Microsoft 365 CSP, SentinelOne, Acronis, etc.) through
Pax8, which handles provisioning via vendor APIs and consolidates everything onto
one bill. Tenant IDs, partner data, and API credentials are **secrets** — keep
real values in the MSP secret store, never in notes.

## What the platform does

| Capability | What it means |
|------------|---------------|
| **Quote → Order** | Browse the marketplace, build orders/quotes in one place |
| **Provision** | ~98.5% automated via vendor APIs; manual cases < ~15 min |
| **Bill** | Proprietary billing engine → one consolidated monthly invoice |
| **Storefront** | Spin up your own branded store so *you* are the client's marketplace |
| **Pax8 Pro** | Multitenant visibility + business insights across customers |
| **PSA integrations** | Sync companies/products/billing into your PSA (300+ tools) |

> [!key-insight] The billing engine is the sticky part. Pax8 normalizes every
> vendor's invoice into one monthly bill with monthly/annual/usage tiers and
> proration — saving the manual reconciliation hours that used to eat an MSP's
> month-end. Combined with the storefront, clients have little reason to source
> cloud elsewhere.

## Microsoft CSP

A primary reason MSPs use Pax8 is as a **Microsoft CSP (Cloud Solution Provider)
indirect** distributor — handling M365/Azure licensing under the New Commerce
Experience (NCE), with its term/seat/commitment rules surfaced in the marketplace.
What the licenses actually deliver is in [[Microsoft 365 Services]].

## Driving Pax8 from a shell (the API)

There is **no official Pax8 CLI binary** — "the Pax shell" in practice means
driving the **Pax8 REST API** from a terminal (cURL / scripts) or via community
modules. It's a standard OAuth 2.0 **client-credentials** flow.

```bash
# 1) Get a bearer token (TTL ~1 day). Audience selects the API surface:
#    api://provisioning  for provisioning endpoints
#    api://usage         for usage endpoints
curl -s -X POST https://api.pax8.com/v1/token \
  -H "Content-Type: application/json" \
  -d '{
        "client_id": "<client-id>",
        "client_secret": "<client-secret>",
        "audience": "api://provisioning",
        "grant_type": "client_credentials"
      }'

# 2) Call the v2 API with the returned token
curl -s https://api.pax8.com/v2/companies \
  -H "Authorization: Bearer <access-token>"
```

- **Credentials**: *Settings → Integrations → Credentials → + Create API
  credential*, then copy the Client ID / Client Secret (store securely).
- **Base URL**: `https://api.pax8.com/v2`; token endpoint `…/v1/token`.
- **Developer platform / reference**: `devx.pax8.com`.

> [!note] Client-credentials is for **your own** internal automation. To access
> *another partner's* data with a third-party app you must use the **Delegated
> OAuth** (authorization-code) flow — the partner explicitly consents first.

### Community tooling

No first-party CLI, but unofficial wrappers make shell use friendlier:

- **PowerShell** — `Pax8-API` / `Pax8API` modules (`Connect-Pax8`,
  `Get-Pax8Company`, `Get-Pax8Product -search 'Microsoft'`, `Get-Pax8Invoice`).
- **Python** — dltHub source for piping Pax8 data into a warehouse/DuckDB.

## Where it fits

- Licensing it sells maps to [[Microsoft 365 Services]].
- It's the procurement/billing backbone of [[Managed IT Services]] delivery —
  including the seats for tools like [[Huntress]] and [[Avanan]] — at
  [[CK Technology LLC]].

## Related

- [[Microsoft 365 Services]] — the CSP licensing Pax8 distributes
- [[Managed IT Services]] — the delivery model Pax8 underpins
- [[Huntress]] · [[Avanan]] — security products often procured through it
