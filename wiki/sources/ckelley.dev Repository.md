---
type: source
title: "ckelley.dev Repository"
source_type: repository
author: "Christopher Kelley (GhostKellz)"
date_published: 2026-06-21
url: "https://ckelley.dev"
confidence: high
key_claims:
  - "Astro 6 static portfolio with zero client-side JS framework"
  - "Architecture page renders infrastructure as hand-authored animated inline SVG"
created: 2026-06-21
updated: 2026-06-21
tags:
  - source
  - repository
  - web
status: seed
related:
  - "[[ckelley.dev]]"
  - "[[Hand-Authored Inline SVG Diagrams]]"
  - "[[Zero-JS Static Site Approach]]"
---

# ckelley.dev Repository

## Summary

The source repository for [[ckelley.dev]], Christopher Kelley's portfolio and
technical-resume site. Built with [[Astro Static Site Generator]] 6, [[Tailwind CSS 4]],
TypeScript, and Vite 7; output is static HTML/CSS served by [[NGINX Static Site Hosting]].
Read at `/data/projects/ckelley.dev` (README + `src/`).

## Key Claims

- No client-side framework or hydration — [[Zero-JS Static Site Approach]].
- The `/architecture` page composes 20+ inline-SVG flow components with CSS-only
  animation — [[Hand-Authored Inline SVG Diagrams]] — covering virtualization, AI,
  security, and observability.
- Deployed by building to `dist/` and `rsync`-ing to an NGINX web root.

## Entities Mentioned

- [[ckelley.dev]] — the website this repo builds.
- [[GhostKellz]] — author/operator.

## Concepts Introduced

- [[Hand-Authored Inline SVG Diagrams]] — the architecture-page diagram technique.
- [[Zero-JS Static Site Approach]] — the site's core thesis.

## Notes

The architecture page also documents homelab/production infrastructure (FortiGate
SD-WAN, Proxmox + VFIO, local AI inference, CrowdSec, Heimdall observability,
Tailscale). Those topics belong to the system/networking and dev-ecosystem domains of
this vault and are intentionally not duplicated here; only the web-publishing
technique is captured under this domain.
