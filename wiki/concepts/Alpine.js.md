---
type: concept
title: "Alpine.js"
complexity: beginner
domain: web
aliases:
  - AlpineJS
created: 2026-06-21
updated: 2026-06-21
tags:
  - concept
  - web
  - javascript
status: seed
related:
  - "[[Astro Static Site Generator]]"
  - "[[Zero-JS Static Site Approach]]"
  - "[[ghostkellz.sh]]"
  - "[[cktechx.com]]"
---

# Alpine.js

## Definition

Alpine.js is a small declarative JavaScript framework for sprinkling interactivity
into otherwise-static HTML using `x-data`, `x-show`, `@click`, and similar
attributes. It is the chosen interaction layer for the two sites that need a little
client behaviour: [[ghostkellz.sh]] and [[cktechx.com]].

## How It Is Used Here

- **[[ghostkellz.sh]]** (Alpine 3.15): copy-to-clipboard buttons, the collapsible
  public-key reveal in the `KeyBlock` component, and the `Terminal` component's
  step-driven verification replay (`Alpine.data('terminal', ...)`). Alpine is
  initialized once in `src/layouts/Layout.astro`.
- **[[cktechx.com]]**: mobile navigation toggle and other small dynamic features.

> [!key-insight]
> Alpine is the deliberate exception to the [[Zero-JS Static Site Approach]]: it adds
> just enough behaviour (copy buttons, reveal toggles, a menu) without pulling in a
> full SPA framework or a hydration runtime. Notably, [[ckelley.dev]] avoids even
> Alpine, animating its diagrams with CSS only.

## Connections

- Layered on top of [[Astro Static Site Generator]] output.
- Powers the interactive pieces of [[GPG Key Verification Workflow]] on
  [[ghostkellz.sh]].

## Sources

- [[ghostkellz.sh Repository]]
- [[cktechx.com Repository]]
