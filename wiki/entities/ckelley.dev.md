---
type: entity
title: "ckelley.dev"
entity_type: website
url: "https://ckelley.dev"
role: "Personal portfolio, technical resume, and infrastructure showcase for Christopher Kelley"
created: 2026-06-21
updated: 2026-06-21
tags:
  - website
  - portfolio
  - astro
status: seed
related:
  - "[[GhostKellz]]"
  - "[[Astro Static Site Generator]]"
  - "[[Hand-Authored Inline SVG Diagrams]]"
  - "[[Zero-JS Static Site Approach]]"
  - "[[ghostkellz.sh]]"
  - "[[cktechx.com]]"
---

# ckelley.dev

## Overview

`ckelley.dev` is the personal portfolio and living technical resume of [[GhostKellz]]
(Christopher Kelley). It is a fast static site that doubles as an infrastructure
showcase, covering professional experience, open-source work, the homelab and
production infrastructure, the local/cloud AI stack, and academic background. The
standout artifact is its `/architecture` page, which renders an entire homelab and
production estate as hand-authored, animated inline-SVG flow diagrams.

## Key Facts

- **Stack:** [[Astro Static Site Generator]] 6, [[Tailwind CSS 4]] (via the
  `@tailwindcss/vite` plugin), TypeScript, Vite 7, pnpm.
- **Output:** Pure static HTML/CSS — see [[Zero-JS Static Site Approach]]. No
  client-side framework, no hydration, no charting library.
- **Hosting:** [[NGINX Static Site Hosting]] serving the built `dist/` out of a
  web root, deployed by `rsync`.
- **Signature feature:** [[Hand-Authored Inline SVG Diagrams]] — 20+ flow
  components under `src/components/diagram/flows/`, with CSS-only animated edges,
  per-domain accent theming, and accessibility built in.
- **Routes:** `index`, `architecture`, `ai`, `lab`, `projects`, `technologies`,
  `education`.
- **Owner/operator:** [[GhostKellz]]; copyright attributed to Christopher Kelley /
  CK Technology LLC.

## The Architecture Page

The `/architecture` page is the richest content on the site. It documents a
single end-to-end environment spanning four domains — virtualization, AI/inference,
security, and observability — presented as coordinate-authored SVG topology and
dataflow diagrams. The high-level concepts it surfaces (FortiGate SD-WAN edge, a
Proxmox cluster with VFIO GPU passthrough, local-first AI inference, distributed
CrowdSec, and Heimdall observability over a Tailscale control plane) are infrastructure
topics owned by other domains of this vault; this page captures only the
*web-publishing technique* used to present them.

> [!key-insight]
> The architecture page deliberately uses **no Mermaid, no D3, and zero JS
> libraries** — every diagram is static SVG with CSS animation. This keeps the
> portfolio's "zero-JS static" thesis intact even for complex, branching graphs.

## Connections

- Built and operated by [[GhostKellz]].
- Shares its Astro + Tailwind static stack with [[ghostkellz.sh]] and [[cktechx.com]].
- The About page of [[cktechx.com]] links here as the founder's personal site.
- Diagram approach documented at [[Hand-Authored Inline SVG Diagrams]].

## Sources

- [[ckelley.dev Repository]]
