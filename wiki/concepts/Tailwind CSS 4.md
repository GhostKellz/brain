---
type: concept
title: "Tailwind CSS 4"
complexity: beginner
domain: web
aliases:
  - Tailwind CSS
  - Tailwind 4
created: 2026-06-21
updated: 2026-06-21
tags:
  - concept
  - web
  - css
  - tailwind
status: seed
related:
  - "[[Astro Static Site Generator]]"
  - "[[Zero-JS Static Site Approach]]"
  - "[[ckelley.dev]]"
  - "[[ghostkellz.sh]]"
  - "[[cktechx.com]]"
---

# Tailwind CSS 4

## Definition

Tailwind CSS 4 is the utility-first CSS framework used for styling across all three
of [[GhostKellz]]'s web properties. Version 4 introduces a CSS-first configuration
model: theme tokens are declared with an `@theme` block directly in CSS rather than
in a JavaScript config file.

## How It Is Used Here

- Integrated through the **`@tailwindcss/vite`** plugin in `astro.config.mjs` — no
  separate PostCSS pipeline.
- Theme tokens and custom classes live in `src/styles/global.css`.
- [[ghostkellz.sh]] keeps a `tailwind.config.mjs` for compatibility/documentation,
  but the active configuration is the CSS-first `@theme`.
- [[cktechx.com]] defines a small brand palette as CSS custom properties
  (`--ck-navy`, `--ck-blue`, etc.) and a handful of component classes (`.btn-primary`,
  `.card`, `.section`).
- [[ckelley.dev]] extends `global.css` with flow-diagram classes including the
  `flow-dash` keyframe that animates [[Hand-Authored Inline SVG Diagrams]].

> [!key-insight]
> Tailwind 4's `@theme`-in-CSS model fits the [[Zero-JS Static Site Approach]]
> cleanly: design tokens, utilities, and bespoke animations all live in one CSS
> file that ships as plain static styles.

## Connections

- Paired with [[Astro Static Site Generator]] via the Vite plugin.
- Provides the animation primitives behind [[Hand-Authored Inline SVG Diagrams]].

## Sources

- [[ckelley.dev Repository]]
- [[ghostkellz.sh Repository]]
- [[cktechx.com Repository]]
