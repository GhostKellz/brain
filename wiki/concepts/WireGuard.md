---
type: concept
title: "WireGuard"
created: 2026-06-21
updated: 2026-06-21
tags:
  - wireguard
  - vpn
  - networking
  - cryptography
status: seed
related:
  - "[[Tailscale]]"
  - "[[Mesh VPN]]"
---

# WireGuard

**WireGuard** is a modern VPN protocol built into the Linux kernel. It is
deliberately minimal: a few thousand lines of code, a single modern cryptographic
suite (no negotiation), and a peer model based on public keys. The result is fast,
auditable, and simple compared to IPsec or OpenVPN.

> [!key-insight]
> WireGuard has **no concept of a "connection"** — it's stateless and
> UDP-based. A peer is just a public key plus a set of `AllowedIPs`. Packets to
> those IPs get encrypted to that peer; that's the whole routing model.

## The peer model

```ini
[Interface]
PrivateKey = <this node's private key>
Address    = 10.x.x.1/24
ListenPort = 51820

[Peer]
PublicKey  = <other node's public key>
AllowedIPs = 10.x.x.2/32        # which IPs route to this peer
Endpoint   = <host>:51820        # where to reach it (optional if it dials us)
```

- **`AllowedIPs`** is dual-purpose: a routing table (outbound) *and* an ACL
  (inbound packets from a peer are only accepted if their source is in its
  AllowedIPs).
- **Cryptokey routing** — the public key *is* the identity; no certificates.

## Why it's the foundation for mesh VPNs

WireGuard does the encrypted transport extremely well but leaves **key
distribution, NAT traversal, and ACL management** to you. Doing that by hand for
N nodes is O(N²) tedium. Tools like [[Tailscale]] (and Headscale) automate exactly
that layer — which is why a [[Mesh VPN]] is "WireGuard + a control plane."

## Strengths

- **Performance** — in-kernel, minimal overhead; often faster than OpenVPN/IPsec.
- **Simplicity** — small attack surface, easy to reason about.
- **Roaming** — peers can change IP/endpoint without dropping the tunnel.
- **Quiet** — doesn't respond to unauthenticated packets (no port scan signature).

## Trade-offs

- No built-in key distribution or user management — bring your own (or use a mesh
  tool).
- Static `AllowedIPs` config is fine for a few peers, painful for many.

## Related

- [[Tailscale]] — automates WireGuard key/ACL management
- [[Mesh VPN]] — the pattern WireGuard underpins
