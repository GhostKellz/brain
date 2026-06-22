---
type: entity
title: "KDE Plasma"
created: 2026-06-21
updated: 2026-06-21
tags:
  - kde
  - desktop-environment
  - wayland
  - linux
status: seed
related:
  - "[[Wayland]]"
  - "[[NVIDIA on Wayland]]"
  - "[[Tiling Window Management]]"
---

# KDE Plasma

**KDE Plasma** is a feature-rich, highly configurable Linux desktop environment.
Its compositor/window manager is **KWin**, which runs as both an X11 and a
[[Wayland]] compositor. Plasma is the desktop where the
[[NVIDIA on Wayland]] interplay matters most for many users.

## Key components

| Component | Role |
|-----------|------|
| **KWin** | Window manager + compositor (X11 and Wayland) |
| **Plasma Shell** | Panels, widgets, system tray, desktop |
| **PowerDevil** | Power management (lid, sleep, brightness) |
| **Spectacle** | Screenshot tool (uses PipeWire portals on Wayland) |
| **KWin scripts** | Extend window behaviour (e.g. tiling via Krohnkite) |

## Wayland on KDE

KWin is a mature Wayland compositor, but on NVIDIA it inherits the proprietary-
driver quirks — notably a frozen-monitor bug that's worked around by switching to
a TTY and back (`Ctrl+Alt+F3` then `Ctrl+Alt+F1`) rather than rebooting. Full
detail in [[NVIDIA on Wayland]].

## Tiling on Plasma

Plasma is a floating DE by default, but KWin scripts add dynamic tiling. The most
common is **Krohnkite**, which brings i3/dwm-style layouts (tile, columns,
monocle) with vim-style focus/move/resize bindings. See
[[Tiling Window Management]] for the workflow and recovery shortcuts.

## Useful default shortcuts

| Action | Keys |
|--------|------|
| Snap left / right | `Meta + Left` / `Meta + Right` |
| Maximize / minimize | `Meta + PgUp` / `Meta + PgDown` |
| Overview | `Meta + W` |
| Grid (desktops) | `Meta + G` |
| Previous / next desktop | `Meta + Ctrl + Left` / `Right` |
| Close window | `Alt + F4` |

## Related

- [[Wayland]] — KWin is its compositor
- [[NVIDIA on Wayland]] — the driver pain points on Plasma
- [[Tiling Window Management]] — Krohnkite tiling on KWin
