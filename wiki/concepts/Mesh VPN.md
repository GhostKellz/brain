---
type: concept
title: "Mesh VPN"
created: 2026-06-21
updated: 2026-06-21
tags:
  - networking
  - vpn
  - wireguard
  - zero-trust
status: seed
related:
  - "[[Tailscale]]"
  - "[[WireGuard]]"
  - "[[Zero Trust Networking]]"
---

# Mesh VPN

A **mesh VPN** connects every device directly to every other device with
encrypted peer-to-peer tunnels, instead of routing all traffic through a central
VPN concentrator (the classic hub-and-spoke model). A lightweight **control
plane** distributes keys and access rules; the actual data flows node-to-node.

> [!key-insight]
> Separate the **control plane** (who's allowed to talk to whom — keys, ACLs)
> from the **data plane** (the encrypted tunnels themselves). The control server
> never sees your traffic; it just brokers identity and policy.

## Control plane vs data plane

```
Control plane:  identity + keys + ACLs  →  hands each node its config
Data plane:     direct encrypted tunnels between nodes (peer-to-peer)
Relay fallback: when a direct path can't be punched through NAT/firewalls
```

- Nodes authenticate to the control server (often via SSO/OIDC).
- The server distributes public keys and access-control rules.
- Nodes then build **direct** encrypted tunnels to each other.
- If NAT/firewalls block a direct path, traffic falls back to a **relay** — most
  flows never touch one.

## Why it beats hub-and-spoke

- **No chokepoint** — traffic between two nodes takes the shortest path, not a
  detour through a central gateway.
- **Resilience** — losing one node doesn't sever everyone else.
- **Lower latency** — direct paths; relays only as fallback.
- **Fine-grained ACLs** — policy is per-identity, enabling [[Zero Trust Networking]].

## Concrete implementations

- [[Tailscale]] — managed control plane over [[WireGuard]]; the most common
  turnkey mesh.
- **Headscale** — self-hosted, open-source reimplementation of Tailscale's
  control server (good for labs/learning).
- Plain [[WireGuard]] — you *can* build a mesh by hand, but you manage all keys
  and ACLs yourself; the mesh-VPN tools exist to automate exactly that.

## Related

- [[Tailscale]] — turnkey mesh
- [[WireGuard]] — the tunnel layer underneath
- [[Zero Trust Networking]] — the access model a mesh enables
