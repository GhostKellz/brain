---
type: concept
title: "Astro Static Site Generator"
complexity: intermediate
domain: web
aliases:
  - Astro
created: 2026-06-21
updated: 2026-06-21
tags:
  - concept
  - web
  - astro
  - static-site
status: seed
related:
  - "[[Zero-JS Static Site Approach]]"
  - "[[Tailwind CSS 4]]"
  - "[[Alpine.js]]"
  - "[[NGINX Static Site Hosting]]"
  - "[[ckelley.dev]]"
  - "[[ghostkellz.sh]]"
  - "[[cktechx.com]]"
---

# Astro Static Site Generator

## Definition

Astro is a web framework that renders components to static HTML at build time and
ships **zero JavaScript by default**. It is the common foundation for all three of
[[GhostKellz]]'s web properties — [[ckelley.dev]], [[ghostkellz.sh]], and
[[cktechx.com]] — each pinned to Astro 6.x.

## How It Is Used Here

All three sites follow the same shape:

- `.astro` components compose pages from a `BaseLayout`/`Layout` shell plus
  reusable pieces (Hero, Navigation, cards, diagram primitives).
- Routing is file-based: one file per route under `src/pages/`.
- Styling is [[Tailwind CSS 4]], wired through the `@tailwindcss/vite` plugin in
  `astro.config.mjs` rather than a PostCSS step.
- Builds emit a static `dist/` directory served by [[NGINX Static Site Hosting]].

A minimal config from [[ckelley.dev]]:

```js
import { defineConfig } from 'astro/config';
import tailwindcss from '@tailwindcss/vite';

export default defineConfig({
  site: 'https://ckelley.dev',
  vite: { plugins: [tailwindcss()] },
});
```

Standard scripts across the repos: `astro dev` (local at `:4321`), `astro build`
(emit `dist/`), `astro preview`.

## Why Astro for These Sites

> [!key-insight]
> Astro lets each site stay almost entirely static while still composing from
> components. Interactivity is added surgically — [[ghostkellz.sh]] and
> [[cktechx.com]] sprinkle in [[Alpine.js]] for copy buttons and the mobile nav,
> while [[ckelley.dev]] avoids client JS frameworks entirely and animates its
> diagrams with CSS only. This is the [[Zero-JS Static Site Approach]] in practice.

## Connections

- Pairs with [[Tailwind CSS 4]] for styling and optionally [[Alpine.js]] for
  interactions.
- Underpins the [[Zero-JS Static Site Approach]] all three sites share.

## Sources

- [[ckelley.dev Repository]]
- [[ghostkellz.sh Repository]]
- [[cktechx.com Repository]]
