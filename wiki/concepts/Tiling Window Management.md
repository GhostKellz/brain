---
type: concept
title: "Tiling Window Management"
created: 2026-06-21
updated: 2026-06-21
tags:
  - tiling
  - kde
  - productivity
  - linux
status: seed
related:
  - "[[KDE Plasma]]"
  - "[[tmux Cheatsheet]]"
---

# Tiling Window Management

A **tiling window manager** automatically arranges windows to fill the screen
without overlap, instead of free-floating windows you place by hand. You navigate,
move, and resize with the keyboard — windows are tiled into layouts (master/stack,
columns, grid, monocle) that the WM maintains for you.

> [!key-insight]
> The point isn't aesthetics — it's that **the keyboard replaces the mouse** for
> window management, and the WM guarantees no wasted space or hunting for a buried
> window. The same vim-style `h/j/k/l` directional model shows up in tiling WMs,
> [[tmux Cheatsheet|tmux panes]], and vim splits.

## Common layouts

| Layout | Shape |
|--------|-------|
| Tile / master-stack | one large "master" + a stack of the rest |
| Columns | windows side-by-side, equal-ish width |
| Monocle | one window fills the screen (others hidden) |
| Grid | even grid of all windows |

## Krohnkite — tiling on KDE Plasma

[[KDE Plasma]] floats by default; **Krohnkite** is a KWin script that adds dynamic
tiling on top. Typical bindings (`Meta` = Super key):

| Action | Keys |
|--------|------|
| Cycle layouts | `Meta + \` |
| Float current window | `Meta + F` |
| Float all (panic) | `Meta + Shift + F` |
| Set window as master | `Meta + Enter` |
| Focus left/down/up/right | `Meta + H/J/K/L` |
| Move window | `Meta + Shift + H/J/K/L` |
| Resize | `Meta + Ctrl + H/J/K/L` |

## The "everything stopped tiling" panic sequence

> [!key-insight]
> When a Krohnkite layout misbehaves (a window stuck fullscreen, nothing tiling),
> three shortcuts fix the vast majority of cases:
> 1. `Meta + Shift + F` — toggle float-all
> 2. `Meta + \` — cycle layouts back to Tile/Columns
> 3. `Meta + F` — float/unfloat the offending window
>
> A single window eating the whole screen is usually just the **Monocle** layout —
> `Meta + \` cycles out of it.

## Why pair it with tmux

A tiling WM tiles GUI windows; [[tmux Cheatsheet|tmux]] tiles terminal panes
*inside* one window. Sharing the `h/j/k/l` directional bindings across both (and
vim) means one navigation reflex everywhere.

## Related

- [[KDE Plasma]] — host desktop for Krohnkite
- [[tmux Cheatsheet]] — the same model for terminal panes
