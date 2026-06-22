---
type: entity
title: "Unbound"
created: 2026-06-21
updated: 2026-06-21
tags:
  - dns
  - unbound
  - networking
  - privacy
status: seed
related:
  - "[[Recursive DNS Resolution]]"
---

# Unbound

**Unbound** is a validating, recursive, caching DNS resolver from NLnet Labs.
It's a common choice for self-hosted DNS because it does full
[[Recursive DNS Resolution]] (walking from the root) with DNSSEC validation,
while staying lightweight and security-focused.

## What it does

- **Recursive resolution** — resolves queries by querying authoritative servers
  directly rather than forwarding to a third party.
- **DNSSEC validation** — cryptographically verifies responses.
- **Caching** — fast repeat lookups.
- **Local data / overrides** — serve internal zones or override specific records.

## Remote control

Unbound exposes a control interface (`unbound-control`) for live stats and cache
operations, secured with its own key/cert pair:

```yaml
remote-control:
  control-enable: yes
  control-interface: 127.0.0.1
  control-port: 8953
  server-key-file: "/etc/unbound/unbound_server.key"
  server-cert-file: "/etc/unbound/unbound_server.pem"
  control-key-file: "/etc/unbound/unbound_control.key"
  control-cert-file: "/etc/unbound/unbound_control.pem"
```

Generate the key/cert pair with `unbound-control-setup`, then:

```bash
unbound-control stats
unbound-control reload
unbound-control flush <name>
```

## Typical role

Often sits *behind* a filtering resolver (Pi-hole, Technitium) as the recursive +
validating backend, so the filter handles blocklists and Unbound handles the
actual recursion. See [[Recursive DNS Resolution]] for the layered pattern.

> [!gap]
> Listen addresses, forward zones, and internal overrides are infrastructure
> detail and live in the private tier.

## Related

- [[Recursive DNS Resolution]] — the concept Unbound implements
