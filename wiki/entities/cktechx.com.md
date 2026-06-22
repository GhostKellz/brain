---
type: entity
title: "cktechx.com"
entity_type: website
url: "https://cktechx.com"
role: "Public marketing site for CK Technology LLC, a managed IT/cloud/security MSP"
created: 2026-06-21
updated: 2026-06-21
tags:
  - website
  - msp
  - marketing
  - astro
status: seed
related:
  - "[[CK Technology Solutions]]"
  - "[[GhostKellz]]"
  - "[[Managed IT Services]]"
  - "[[Cloud and Hosting Services]]"
  - "[[Network and Security Services]]"
  - "[[Microsoft 365 Services]]"
  - "[[MSP Positioning and Values]]"
  - "[[Astro Static Site Generator]]"
---

# cktechx.com

## Overview

`cktechx.com` is the public marketing website for [[CK Technology Solutions]] (CK
Technology LLC), a managed IT, cloud, and security services provider founded by
[[GhostKellz]]. The site presents the company's services, positioning, founder
story, and contact channels. It is a separate concern from the organization itself:
this page documents the *website*; [[CK Technology Solutions]] documents the *business*.

## Key Facts

- **Stack:** [[Astro Static Site Generator]] 6, [[Tailwind CSS 4]] (via
  `@tailwindcss/vite`), [[Alpine.js]] for the mobile nav and dynamic bits,
  TypeScript, pnpm. Fonts: Inter. Icons: inline SVG (lucide-static).
- **Output:** Pure static HTML/CSS/JS — see [[Zero-JS Static Site Approach]].
- **Hosting:** [[NGINX Static Site Hosting]]; deployed via `rsync` of `dist/`.
- **Contact form:** Formspree backend.
- **Public domains:** main site `cktechx.com`; help portal `help.cktechx.com`.
- **Service region:** New Hampshire and the wider New England area (NH, MA, RI, ME).
- **Brand system:** navy (`#163B63`) + blue accent (`#3FA9F5`) on a light background.

## Pages & Routes

| Route | Purpose |
|-------|---------|
| `/` | Homepage: hero, services overview, CTAs |
| `/about` | Company mission, founder, values, region — see [[MSP Positioning and Values]] |
| `/services` | Overview grid of all services |
| `/services/managed-it` | [[Managed IT Services]] |
| `/services/cloud-hosting` | [[Cloud and Hosting Services]] |
| `/services/security` | [[Network and Security Services]] |
| `/services/microsoft-365` | [[Microsoft 365 Services]] |
| `/contact` | Formspree form + contact info |
| `/privacy` | Privacy policy |
| `/help` | Client help portal (served at `help.cktechx.com`) |

## The Help Portal

The `/help` page targets the `help.cktechx.com` subdomain. It links to a remote-support
endpoint and supports a `?client=` URL parameter to show a personalized welcome banner.

> [!gap]
> Specific client identifiers, internal documentation tooling, and any non-public
> support infrastructure are deliberately excluded from this vault (it is public).
> Only the marketing-level structure of the help portal is recorded here.

## Connections

- Marketing front for [[CK Technology Solutions]] (the org).
- Founded and operated by [[GhostKellz]]; the About page links to [[ckelley.dev]].
- Shares the Astro + Tailwind static stack with [[ckelley.dev]] and [[ghostkellz.sh]].

## Sources

- [[cktechx.com Repository]]
