---
type: source
title: "cktechx.com Repository"
source_type: repository
author: "Christopher Kelley (GhostKellz)"
date_published: 2026-06-21
url: "https://cktechx.com"
confidence: high
key_claims:
  - "Public marketing site for CK Technology LLC, a managed IT/cloud/security MSP"
  - "Astro 6 + Tailwind 4 + Alpine.js static site, NGINX-hosted"
created: 2026-06-21
updated: 2026-06-21
tags:
  - source
  - repository
  - msp
status: seed
related:
  - "[[cktechx.com]]"
  - "[[CK Technology LLC]]"
  - "[[MSP Positioning and Values]]"
---

# cktechx.com Repository

## Summary

The source repository for [[cktechx.com]], the marketing site for
[[CK Technology LLC]] (CK Technology LLC). Built with [[Astro Static Site
Generator]] 6, [[Tailwind CSS 4]], and [[Alpine.js]]; static output served by
[[NGINX Static Site Hosting]] (config in `deploy/cktechx.conf`). Read at
`/data/projects/cktechx.com` (README + `src/`).

## Key Claims

- Four core service lines: [[Managed IT Services]], [[Cloud and Hosting Services]],
  [[Network and Security Services]], [[Microsoft 365 Services]].
- Positioning: founder-led, security-first MSP serving New England SMBs — see
  [[MSP Positioning and Values]].
- Main site `cktechx.com` plus a `help.cktechx.com` help portal; contact form via
  Formspree.

## Entities Mentioned

- [[cktechx.com]] — the website this repo builds.
- [[CK Technology LLC]] — the MSP organization it markets.
- [[GhostKellz]] — founder.

## Concepts Introduced

- The four MSP service concepts plus [[MSP Positioning and Values]].

## Notes

Per vault tiering, only public marketing content is captured. The README references
operational details (server paths, TLS cert locations, remote-support and client-doc
subdomains, a `?client=` personalization parameter, contact phone/email). Internal
infrastructure paths, client identifiers, and any non-public tooling are deliberately
excluded from the vault; public contact channels and service descriptions are the only
operational facts retained, and even those are kept on the entity page rather than
expanded here.
