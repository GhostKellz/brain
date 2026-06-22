---
type: entity
title: "ghostkellz.sh"
entity_type: website
url: "https://ghostkellz.sh"
role: "Official GhostKellz GPG identity and public-key verification site"
created: 2026-06-21
updated: 2026-06-21
tags:
  - website
  - identity
  - gpg
  - astro
status: seed
related:
  - "[[GhostKellz]]"
  - "[[PGP and GPG]]"
  - "[[Web Key Directory]]"
  - "[[GPG Key Verification Workflow]]"
  - "[[Astro Static Site Generator]]"
  - "[[ckelley.dev]]"
---

# ghostkellz.sh

## Overview

`ghostkellz.sh` is a compact static identity site for publishing and verifying the
official [[GhostKellz]] GPG public key. It presents the key fingerprint, a download
link to the armored key, copyable import/verification commands, key metadata, and a
terminal-style signature-verification animation. The site exists so that anyone can
independently confirm that a signature or release genuinely comes from GhostKellz.

## Key Facts

- **Stack:** [[Astro Static Site Generator]] 6, [[Tailwind CSS 4]], [[Alpine.js]] 3
  for small interactions (copy buttons, key reveal, terminal replay), Vite 7.
- **Output:** Static HTML/CSS/JS; assets served from `public/`, including the armored
  key at `public/ghostkellz_pubkey.asc`.
- **Identity (UID):** `GhostKellz <ckelley@ghostkellz.sh>`.
- **Key fingerprint:** `478D3EFD1D9694F6BAD0AC1F777538754BA2B57D` (public — safe to
  reference).
- **Key ID:** `777538754BA2B57D`.
- **Algorithm:** Ed25519 (signing) / Cv25519 (encryption).
- **Created:** 2025-04-11; **Expires:** 2027-04-05.
- **Discovery:** [[Web Key Directory]] (WKD) / DANE.

> [!key-insight]
> The whole point of the site is *out-of-band trust*: by serving the key over HTTPS
> on the same domain as the email address, [[Web Key Directory]] lets `gpg
> --locate-keys ckelley@ghostkellz.sh` fetch and bind the key automatically.

## Site Features

- **KeyBlock** component — fingerprint display, key download, and collapsible armored
  key viewer with copy actions.
- **Terminal** component — an [[Alpine.js]]-driven replay of a `gpg --locate-keys`
  lookup followed by a `gpg --verify` "Good signature" result.
- **GhostFetch** card — a `neofetch`-style daily-driver terminal card.
- Verification quadrant documenting WKD import, URL import, signature verify, and
  fingerprint check — see [[GPG Key Verification Workflow]].

## Connections

- The identity belongs to [[GhostKellz]].
- Links out to GitHub orgs `ghostkellz` and `CK-Technology` (the latter tied to
  [[CK Technology LLC]]).
- Shares the Astro + Tailwind static stack with [[ckelley.dev]] and [[cktechx.com]].

## Sources

- [[ghostkellz.sh Repository]]
