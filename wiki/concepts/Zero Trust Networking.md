---
type: concept
title: "Zero Trust Networking"
created: 2026-06-21
updated: 2026-06-21
tags:
  - networking
  - security
  - zero-trust
  - vpn
status: seed
related:
  - "[[Mesh VPN]]"
  - "[[Tailscale]]"
  - "[[nftables Firewall]]"
---

# Zero Trust Networking

**Zero trust** discards the idea of a trusted internal network. There is no
"inside" that is automatically safe; every access is authenticated and authorized
by identity, regardless of where the request originates. The mantra: **never
trust, always verify**.

> [!key-insight]
> The old model trusted the LAN — once you were "inside the perimeter" you could
> reach most things. Zero trust replaces the perimeter with per-identity,
> per-resource policy, so a compromised device can't roam laterally.

## Core principles

- **Identity-based access** — policy is attached to *who/what* is connecting, not
  *where from*.
- **Least privilege** — each identity gets only the specific resources it needs.
- **Assume breach** — design so a compromised node has minimal blast radius.
- **Encrypt everywhere** — including east-west (internal) traffic.

## How a homelab/MSP realizes it

A practical pattern using a [[Mesh VPN]]:

1. Every device joins an encrypted overlay ([[Tailscale]] / Headscale).
2. Management traffic (SSH, dashboards, APIs) rides the overlay, so it **never
   crosses the public internet** — even between sites.
3. **ACLs** on the control plane express "identity X may reach service Y" — least
   privilege at the network layer.
4. SSO/OIDC ties overlay membership to a real identity provider.
5. A host firewall ([[nftables Firewall]]) provides defense-in-depth per machine.

## What it is not

- It's not "a product you buy." It's an architecture; mesh VPNs, host firewalls,
  SSO, and ACLs are *components* that implement it.
- It's not "no VPN" — it's a shift from a flat trusted tunnel to per-identity,
  per-resource authorization.

> [!gap]
> Specific ACL rules, identity providers, and topology are infrastructure detail
> and live in the private tier.

## Related

- [[Mesh VPN]] — the connectivity substrate
- [[Tailscale]] — ACLs + SSO in practice
- [[nftables Firewall]] — per-host defense-in-depth
