---
type: concept
title: "Hand-Authored Inline SVG Diagrams"
complexity: advanced
domain: web
aliases:
  - Inline SVG flow diagrams
  - CSS-animated SVG diagrams
created: 2026-06-21
updated: 2026-06-21
tags:
  - concept
  - web
  - svg
  - dataviz
  - accessibility
status: seed
related:
  - "[[Zero-JS Static Site Approach]]"
  - "[[Astro Static Site Generator]]"
  - "[[Tailwind CSS 4]]"
  - "[[ckelley.dev]]"
---

# Hand-Authored Inline SVG Diagrams

## Definition

This is the diagramming technique behind the [[ckelley.dev]] `/architecture` page:
infrastructure topology and dataflows are rendered as **coordinate-authored inline
SVG**, animated with CSS only — no Mermaid, no D3, and no charting library. There are
20+ such flow components under `src/components/diagram/flows/`.

## How It Works

The page composes three reusable primitives plus per-flow components:

- **`DiagramFrame.astro`** — a titled, toned frame with a legend slot and optional
  horizontal scroll for wide diagrams.
- **`DiagramLane.astro`** — a lane-based stacked primitive (rows of nodes with
  connectors).
- **`DiagramNode.astro`** — the reusable node primitive (icon, title, sub-lines,
  tone, glow).

Each flow component (e.g. `FortiGateSdwanFlow.astro`) declares its nodes as typed
data — `{ x, y, w, title, subs }` — and computes edge paths with small helper
functions that emit cubic Bézier `M … C …` SVG path strings between node anchor
points:

```ts
const vpath = (a, b) =>
  `M ${a.x} ${a.y} C ${a.x} ${(a.y+b.y)/2}, ${b.x} ${(a.y+b.y)/2}, ${b.x} ${b.y}`;
```

This produces true branching/merging graphs (a node can fan out to several children
and they can merge again), not just stacked boxes.

## Key Properties

- **CSS-only animation:** a `flow-dash` keyframe (in `global.css`) animates a
  "flowing dash" along edges; terminal nodes glow. Honors `prefers-reduced-motion`.
- **Per-domain theming:** `--node` / `--edge` CSS custom properties accent each
  domain (teal/blue/violet/red/emerald/amber).
- **Edge semantics:** solid edges = data path; dotted edges = observe-only / SLA /
  feedback signals, explained in each diagram's legend.
- **Scoped defs:** each diagram carries its own `<defs>` with a uniquely-suffixed
  arrowhead marker so multiple diagrams coexist on one page without ID collisions.
- **Responsive:** `viewBox` scaling with horizontal scroll for wide diagrams on
  mobile.
- **Accessible:** `role="img"` with `<title>`/`<desc>`; decorative markers are
  `aria-hidden`.

> [!key-insight]
> By authoring SVG by hand and animating with CSS, the architecture page renders
> genuinely complex graphs while keeping the [[Zero-JS Static Site Approach]] intact —
> there is no client-side rendering step at all. The trade-off is that coordinates are
> authored to match real topology rather than auto-laid-out.

## Connections

- A concrete expression of the [[Zero-JS Static Site Approach]] on [[ckelley.dev]].
- Built as [[Astro Static Site Generator]] components and styled with
  [[Tailwind CSS 4]].

## Sources

- [[ckelley.dev Repository]]
