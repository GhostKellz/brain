---
type: concept
title: "Internal Domain Naming"
created: 2026-06-28
updated: 2026-06-28
aliases:
  - "Why not .local"
  - ".local domain"
  - "RFC 6762"
  - "AD domain naming"
tags:
  - dns
  - active-directory
  - networking
  - mdns
  - naming
status: seed
related:
  - "[[Active Directory Administration]]"
  - "[[Recursive DNS Resolution]]"
  - "[[Microsoft 365 Services]]"
  - "[[acme.sh - DNS-01 Certificates]]"
  - "[[Networking Reference]]"
---

# Internal Domain Naming

What you name an internal network or **Active Directory** domain is a decision you
live with for the life of the environment — renaming an AD forest is painful. The
single most important rule: **don't use `.local`** (or any made-up TLD). Use a
**subdomain of a public domain you actually own** (e.g. `ad.example.com`,
`internal.example.com`).

> [!key-insight]
> `.local` isn't "yours" — it's **reserved by [RFC 6762] for Multicast DNS
> (mDNS/Bonjour)**. When you name an AD domain `company.local`, you're squatting on
> the protocol macOS, Linux (Avahi), and printers use for zero-config discovery. In
> mixed-OS networks those clients **send `.local` lookups to multicast instead of
> your DNS server**, so resolution silently fails or races. Using `.local` for AD
> is now widely discouraged and effectively deprecated practice.

## What RFC 6762 actually reserves

**Multicast DNS** lets devices resolve names on a local link *without* a DNS
server: a query for `printer.local` is sent to the multicast group `224.0.0.251`
(`ff02::fb` for IPv6) on UDP **5353**, and the owning host answers directly. RFC
6762 formally reserves the **`.local`** top-level domain for exactly this. Apple
Bonjour, Avahi (Linux), and Windows all implement it.

The collision: an OS that sees a `.local` name may **route it to mDNS multicast by
policy**, never consulting your unicast AD DNS. You get intermittent failures that
look like "DNS is broken" but are really a namespace conflict you can't win.

## Why `.local` (and made-up TLDs) hurt

| Problem | What breaks |
|---------|-------------|
| **mDNS/Bonjour conflict** | Macs/Linux/printers route `.local` to multicast, not your DNS → intermittent resolution failures in mixed-OS shops. |
| **Public TLS certs** | Public CAs **cannot** issue certs for non-globally-unique names. No Let's Encrypt for `*.company.local` — you're forced into an internal CA. |
| **Cloud / federation / SSO** | [[Microsoft 365 Services|Entra ID]], hybrid join, and SSO expect **publicly resolvable** UPNs/domains. `.local` makes hybrid integration a fight (UPN suffix workarounds, etc.). |
| **Future TLD conflicts** | Made-up TLDs (`.lan`, `.corp`, `.home`) risk colliding if ICANN ever delegates them. Only an owned name is conflict-proof. |

> [!warning]
> `.corp`, `.home`, and `.mail` were specifically flagged as **high-risk** by
> ICANN/SSAC for name-collision and were withheld from delegation precisely because
> so many internal networks squatted on them. "It's reserved-ish" is not a
> guarantee — own the name.

## The reserved / special-use TLD landscape

Not every made-up TLD is equal. Know which names are *actually* reserved and what
each is for:

| TLD | Status | Use for an AD/internal domain? |
|-----|--------|-------------------------------|
| `.local` | **RFC 6762** — reserved for **mDNS** | **No** — collides with Bonjour/Avahi |
| `.localhost` | RFC 6761 — loopback only | No |
| `.test` `.example` `.invalid` | RFC 6761 — testing/docs/sentinel | No (not for real infra) |
| `.internal` | **ICANN-designated (2024)** for **private-use** networks | **OK** if you want a guaranteed-never-delegated private TLD |
| `.corp` `.home` `.mail` | High-risk, **permanently withheld** by ICANN | No — collision risk |
| any other made-up TLD (`.lan`, `.lab`, `.ad`, …) | **Unreserved** — could be delegated | No — future ownership conflict |
| `ad.example.com` (owned subdomain) | Globally unique, **you own it** | **Best practice** |

> [!key-insight]
> Since 2024 there are exactly **two safe non-public choices** and one best
> practice: (1) **own a public domain and use a private subdomain**
> (`ad.example.com`) — best, because you also get **public TLS certs**; or (2) use
> **`.internal`**, the TLD ICANN reserved specifically so private networks have a
> name guaranteed never to be delegated. Everything else (`.local`, `.corp`,
> `.home`, `.lan`, …) is either someone else's protocol or an unreserved gamble.

> [!note]
> `.internal` solves *collision* and *future-ownership* but **not** the
> certificate problem — public CAs still can't issue for it (it isn't globally
> unique), so you'd run an internal CA. The owned-subdomain route is the only one
> that gives you real, publicly-trusted certs via DNS-01.

## The recommended pattern

Pick a **registered public domain you control** and carve out a subdomain used
*only* internally:

```
example.com            ← public, registered, you own it
├── www.example.com    ← public website
├── ad.example.com     ← Active Directory domain  (internal only)
│   ├── dc01.ad.example.com
│   └── srv01.ad.example.com
└── internal.example.com ← other internal services
```

- **AD forest root:** `ad.example.com` (or `corp.example.com` / `int.example.com`).
- **Split-horizon DNS:** the internal resolver answers `ad.example.com` privately;
  the public authoritative zone never publishes those records. See
  [[Recursive DNS Resolution]].
- **Public certs just work:** you can prove ownership of `example.com` via
  **DNS-01** and issue real certs for internal hostnames without exposing them —
  see [[acme.sh - DNS-01 Certificates]].
- **Cloud-ready:** UPNs become `user@example.com`, matching your
  [[Microsoft 365 Services|M365/Entra]] tenant — clean hybrid identity.

> [!note]
> Using a real FQDN does **not** mean publishing internal hosts to the world.
> Split-horizon / internal-only zones keep `ad.example.com` records private; you're
> just borrowing a globally-unique *name*, not exposing the *hosts*.

## AD-specific: when to use a real FQDN

For a **new** on-prem or Proxmox-hosted AD forest, the decision is essentially
made for you — use a real FQDN (owned subdomain). The only real fork is *which*
non-public fallback you tolerate if you genuinely have no domain to spare:

| Scenario | Forest root to pick |
|----------|--------------------|
| You own a public domain (almost always) | **`ad.example.com`** — owned subdomain. Gets you public certs + clean hybrid identity. |
| Truly air-gapped, will *never* touch cloud/public certs, and don't want to register anything | **`ad.internal`** — ICANN private-use TLD, collision-proof, but internal CA only. |
| Anything else (`.local`, `.corp`, `.lan`, `.ad`) | **Don't.** mDNS collision or future-delegation gamble. |

> [!key-insight]
> The question "should AD use a real FQDN?" has effectively one modern answer:
> **yes, an owned subdomain**, unless you are *certain* the forest will never need
> a publicly-trusted certificate or hybrid (Entra/M365) identity. The moment either
> of those enters the picture — and for most environments they eventually do — a
> real FQDN is the only choice that doesn't force a painful forest rebuild later.
> `.internal` is the *only* defensible non-owned fallback, and even then you accept
> an internal CA forever.

## When `.local`-style names are fine

mDNS itself is great for what it's *for*: a laptop finding a printer or a Chromecast
on the same link with **zero config**. Leave `.local` to Bonjour/Avahi. The mistake
is only **overloading `.local` as your managed unicast/AD namespace**.

## Already stuck on `.local`?

Renaming an AD domain is disruptive (and `Rendom`/domain rename is unsupported with
Exchange and many modern setups). Practical mitigations until a rebuild:

- Add a **routable UPN suffix** (`example.com`) to the AD forest so user logons /
  M365 sync use a real name even if the AD DNS name stays `.local`.
- Run an **internal CA** for `.local` certs (the tax you pay for the legacy name).
- Plan a greenfield migration to `ad.example.com` for the next forest.

## Related

- [[Active Directory Administration]] — where the forest name is chosen (and stuck)
- [[Recursive DNS Resolution]] — split-horizon / internal-only zones
- [[acme.sh - DNS-01 Certificates]] — real certs for internal names via DNS-01
- [[Microsoft 365 Services]] — why hybrid identity wants a routable domain
- [[Networking Reference]] — mDNS (5353) vs unicast DNS on the wire

[RFC 6762]: https://www.rfc-editor.org/rfc/rfc6762
