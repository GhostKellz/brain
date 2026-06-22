---
type: concept
title: "PGP and GPG"
complexity: intermediate
domain: security
aliases:
  - PGP
  - GPG
  - GnuPG
  - OpenPGP
created: 2026-06-21
updated: 2026-06-21
tags:
  - concept
  - security
  - cryptography
  - identity
status: seed
related:
  - "[[Web Key Directory]]"
  - "[[GPG Key Verification Workflow]]"
  - "[[ghostkellz.sh]]"
  - "[[GhostKellz]]"
---

# PGP and GPG

## Definition

PGP (Pretty Good Privacy) is a standard — formalized as OpenPGP — for signing and
encrypting data using public-key cryptography. GPG (GnuPG) is the free implementation
of that standard. A keypair has a private half (kept secret) and a public half
(published so others can verify signatures and send encrypted messages). This concept
is documented in this vault because it is the entire subject of [[ghostkellz.sh]].

## The GhostKellz Key

[[ghostkellz.sh]] publishes the official [[GhostKellz]] OpenPGP identity:

| Field | Value |
|-------|-------|
| UID | `GhostKellz <ckelley@ghostkellz.sh>` |
| Fingerprint | `478D3EFD1D9694F6BAD0AC1F777538754BA2B57D` |
| Key ID | `777538754BA2B57D` |
| Algorithm | Ed25519 (sign) / Cv25519 (encrypt) |
| Created | 2025-04-11 |
| Expires | 2027-04-05 |
| Discovery | [[Web Key Directory]] / DANE |

> [!key-insight]
> The **fingerprint** is the thing that matters for trust. It is a hash of the public
> key, short enough to compare by eye or read aloud. Two parties who confirm the same
> fingerprint over an independent channel can trust everything that key signs. The
> fingerprint and key ID are public by design — only the private key must stay secret.

Ed25519/Cv25519 (the Curve25519 family) is chosen for modern elliptic-curve
signatures and encryption: small keys, fast operations, strong security.

## Why Publish a Key Site At All

A dedicated, HTTPS-served identity page gives people an authoritative place to fetch
the key and confirm the fingerprint before trusting a signed release, email, or
commit. It turns "is this really from GhostKellz?" into a one-command check — see
[[GPG Key Verification Workflow]].

## Connections

- Distribution/discovery handled by [[Web Key Directory]].
- Day-to-day usage captured in [[GPG Key Verification Workflow]].
- The identity belongs to [[GhostKellz]] and is published on [[ghostkellz.sh]].

## Sources

- [[ghostkellz.sh Repository]]
