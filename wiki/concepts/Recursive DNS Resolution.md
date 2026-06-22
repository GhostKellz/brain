---
type: concept
title: "Recursive DNS Resolution"
created: 2026-06-21
updated: 2026-06-21
tags:
  - dns
  - networking
  - unbound
  - privacy
status: seed
related:
  - "[[Unbound]]"
  - "[[Zero Trust Networking]]"
---

# Recursive DNS Resolution

A **recursive resolver** answers a DNS query by doing the legwork itself: it walks
the DNS hierarchy from the root servers down, rather than forwarding your query to
a third-party resolver (like a public 8.8.8.8 / 1.1.1.1). The alternative — a
**forwarding** resolver — just relays to an upstream and caches the answer.

> [!key-insight]
> Recursive = "I'll go ask the authoritative servers myself." Forwarding = "I'll
> ask someone else and pass on their answer." Running your own recursive resolver
> means no third party logs every domain you visit.

## How recursion works

```
client → your resolver → root servers     ("who handles .com?")
                      → TLD servers        ("who handles example.com?")
                      → authoritative NS   ("what's the A record for www?")
       ← cached + returned to client
```

The resolver caches each step, so subsequent lookups are fast and don't re-walk
the tree.

## Why run your own (e.g. Unbound)

- **Privacy** — your browsing isn't funneled through a single upstream provider.
- **DNSSEC validation** — verify answers cryptographically, locally.
- **Control** — local overrides, split-horizon answers, custom zones.
- **No single point of trust** — you talk to the authoritative source directly.

## Root hints

A recursive resolver bootstraps from a **root hints** file (the IPs of the root
servers). It must be kept reasonably current; resolvers like [[Unbound]] ship
with one and it can be refreshed periodically.

## Common layered setups

A frequent homelab pattern chains a filtering layer in front of a recursive one:

```
client → ad/tracker filter (Pi-hole / Technitium) → Unbound (recursive + DNSSEC) → root
```

The filter blocks unwanted domains; Unbound does the actual recursion and
validation. This pairs naturally with [[Zero Trust Networking]] where internal
names resolve only on the private overlay.

> [!gap]
> Specific resolver IPs, internal zones, and forwarder topology are
> infrastructure detail and live in the private tier.

## Related

- [[Unbound]] — a common validating recursive resolver
- [[Zero Trust Networking]] — internal name resolution on the overlay
