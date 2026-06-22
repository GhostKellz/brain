---
type: concept
title: "GPG Key Verification Workflow"
complexity: beginner
domain: security
aliases:
  - GPG verification
  - Signature verification workflow
created: 2026-06-21
updated: 2026-06-21
tags:
  - concept
  - security
  - gpg
  - workflow
status: seed
related:
  - "[[PGP and GPG]]"
  - "[[Web Key Directory]]"
  - "[[ghostkellz.sh]]"
  - "[[Alpine.js]]"
---

# GPG Key Verification Workflow

## Definition

This is the concrete set of commands [[ghostkellz.sh]] documents for fetching the
[[GhostKellz]] public key and verifying that a file or release is genuinely signed by
it. The site presents these as copyable snippets plus an animated terminal that
replays the happy path.

## The Four Steps

1. **Import via [[Web Key Directory]]** — fetch and bind the key from the domain:
   ```bash
   gpg --locate-keys ckelley@ghostkellz.sh
   ```
2. **Import from URL** (fallback) — pull the armored key directly:
   ```bash
   curl -sL https://ghostkellz.sh/ghostkellz_pubkey.asc | gpg --import
   ```
3. **Verify a signature** — confirm a file's detached signature:
   ```bash
   gpg --verify file.sig file
   ```
4. **Check the fingerprint** — compare against the published value:
   ```bash
   gpg --fingerprint ckelley@ghostkellz.sh
   ```
   Expected: `478D3EFD1D9694F6BAD0AC1F777538754BA2B57D`.

## The Animated Terminal

The site's `Terminal` component ([[Alpine.js]]-driven) replays a locate-keys lookup
that prints the key block, then a `gpg --verify` that ends in the canonical success
line:

```
gpg: Good signature from "GhostKellz <ckelley@ghostkellz.sh>"
```

> [!key-insight]
> The trust hinge is step 4. Importing a key only means you *have* a key; confirming
> the **fingerprint** against the value published on the HTTPS site (or compared
> out-of-band) is what proves it is the right one. A "Good signature" line is only
> meaningful once the fingerprint is confirmed.

## Connections

- Built on [[PGP and GPG]] and [[Web Key Directory]].
- Interactive pieces powered by [[Alpine.js]] on [[ghostkellz.sh]].

## Sources

- [[ghostkellz.sh Repository]]
