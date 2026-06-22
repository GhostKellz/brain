---
type: entity
title: "Tailscale"
created: 2026-06-21
updated: 2026-06-21
tags:
  - tailscale
  - networking
  - vpn
  - wireguard
status: seed
related:
  - "[[Mesh VPN]]"
  - "[[WireGuard]]"
  - "[[Zero Trust Networking]]"
  - "[[Tailscale Operations]]"
---

# Tailscale

**Tailscale** is a [[Mesh VPN]] built on [[WireGuard]] with a managed control
plane. It creates a private overlay network (a "tailnet") where every enrolled
device gets a stable address and can reach every other device through direct,
encrypted peer-to-peer tunnels — without manual key management or port forwarding.

> [!key-insight]
> Tailscale's value is everything *around* WireGuard: identity (SSO/OIDC),
> automatic key distribution and rotation, NAT traversal, ACLs, and relay
> fallback. WireGuard is the tunnel; Tailscale is the coordination.

## What it provides

- **Coordination server** — distributes keys and ACLs; never sees your traffic.
- **NAT traversal** — punches direct paths through most NATs/firewalls.
- **DERP relays** — fallback path when a direct connection can't be established;
  most traffic never touches one.
- **MagicDNS** — names instead of IPs across the tailnet.
- **ACLs** — per-identity access policy enabling [[Zero Trust Networking]].
- **SSO/OIDC login** — tie tailnet membership to an identity provider.

## Headscale — the self-hosted control plane

**Headscale** is an open-source reimplementation of Tailscale's control server.
Paired with the normal Tailscale client, it lets you run the coordination plane
yourself — useful for labs and learning the moving parts. The data plane (the
WireGuard tunnels) is identical either way. See [[Headscale Self-Hosting]] for
the full self-hosting runbook and [[Tailscale Operations]] for the client side.

## Useful operational notes

```bash
# Trayscale and other GUIs need you registered as the local operator
sudo tailscale set --operator=$USER

# Firewall: allow the Tailscale UDP port inbound for direct connections,
# or it falls back to DERP relays (see nftables Firewall)
# udp dport 41641 accept
```

## Where it fits a homelab/MSP

A mesh control plane lets management traffic (SSH, dashboards, APIs) ride the
encrypted overlay so it never crosses the public internet — a core
[[Zero Trust Networking]] pattern. Self-hosted Headscale is typically kept to a
lab role rather than production.

> [!gap]
> Specific tailnet topology, node names, relay locations, and ACL contents are
> infrastructure detail and live in the private tier.

## Related

- [[Mesh VPN]] — the general pattern
- [[WireGuard]] — the underlying tunnel protocol
- [[Zero Trust Networking]] — the access model
