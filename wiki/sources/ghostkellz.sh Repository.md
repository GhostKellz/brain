---
type: source
title: "ghostkellz.sh Repository"
source_type: repository
author: "Christopher Kelley (GhostKellz)"
date_published: 2026-06-21
url: "https://ghostkellz.sh"
confidence: high
key_claims:
  - "Static GPG identity site publishing the GhostKellz public key for WKD discovery"
  - "Astro 6 + Tailwind 4 + Alpine.js 3"
created: 2026-06-21
updated: 2026-06-21
tags:
  - source
  - repository
  - security
status: seed
related:
  - "[[ghostkellz.sh]]"
  - "[[PGP and GPG]]"
  - "[[Web Key Directory]]"
  - "[[GPG Key Verification Workflow]]"
---

# ghostkellz.sh Repository

## Summary

The source repository for [[ghostkellz.sh]], the GhostKellz GPG identity and key
verification site. Built with [[Astro Static Site Generator]] 6, [[Tailwind CSS 4]],
and [[Alpine.js]] 3 for small interactions. Read at `/data/projects/ghostkellz.sh`
(README + `src/`).

## Key Claims

- Publishes the [[GhostKellz]] OpenPGP key: fingerprint
  `478D3EFD1D9694F6BAD0AC1F777538754BA2B57D`, Ed25519/Cv25519, created 2025-04-11,
  expires 2027-04-05.
- Key discovery via [[Web Key Directory]] (WKD) / DANE; armored key also served at
  `/ghostkellz_pubkey.asc`.
- Provides copyable import/verify commands and an Alpine-driven terminal replay — the
  [[GPG Key Verification Workflow]].

## Entities Mentioned

- [[ghostkellz.sh]] — the website this repo builds.
- [[GhostKellz]] — the identity being published.

## Concepts Introduced

- [[PGP and GPG]], [[Web Key Directory]], [[GPG Key Verification Workflow]].

## Notes

The repo embeds the full armored public key block in `src/components/KeyBlock.astro`.
That block is public by design, but per vault tiering only the fingerprint/key ID are
referenced in wiki pages — the full key block is not pasted into the vault.
