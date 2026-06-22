---
type: concept
title: "Web Key Directory"
complexity: intermediate
domain: security
aliases:
  - WKD
created: 2026-06-21
updated: 2026-06-21
tags:
  - concept
  - security
  - cryptography
  - gpg
status: seed
related:
  - "[[PGP and GPG]]"
  - "[[GPG Key Verification Workflow]]"
  - "[[ghostkellz.sh]]"
---

# Web Key Directory

## Definition

Web Key Directory (WKD) is a standard for discovering someone's OpenPGP public key
directly from the domain in their email address, over HTTPS. Instead of trusting a
public keyserver, a client looks up the key at a well-known location on the domain
that controls the address — so the same party that owns the email also vouches for the
key. [[ghostkellz.sh]] publishes the [[GhostKellz]] key for WKD discovery.

## How It Works in Practice

The user-facing payoff is a single command — given an address, GPG fetches and binds
the right key automatically:

```bash
gpg --locate-keys ckelley@ghostkellz.sh
```

Because the lookup is anchored to `ghostkellz.sh` (the domain in the UID), a successful
fetch means the key was served by the domain owner over TLS. [[ghostkellz.sh]] also
offers a direct-URL fallback that bypasses WKD:

```bash
curl -sL https://ghostkellz.sh/ghostkellz_pubkey.asc | gpg --import
```

The site lists the key's discovery method as **WKD / DANE** — DANE being the
complementary mechanism that publishes key material via DNSSEC-signed records.

> [!key-insight]
> WKD shifts trust from anonymous keyservers (where anyone can upload a key for any
> address) to the **domain owner**. For a self-hosted identity like
> [[ghostkellz.sh]], this is ideal: the person who controls the website and the email
> is the same person who controls the key.

## Connections

- A distribution mechanism for [[PGP and GPG]] keys.
- The first step in the [[GPG Key Verification Workflow]] published on
  [[ghostkellz.sh]].

## Sources

- [[ghostkellz.sh Repository]]
