---
type: concept
title: "Wayland"
created: 2026-06-21
updated: 2026-06-21
tags:
  - wayland
  - linux
  - display-server
status: seed
related:
  - "[[NVIDIA on Wayland]]"
  - "[[KDE Plasma]]"
---

# Wayland

**Wayland** is the modern Linux display server protocol, the successor to X11. In
Wayland the **compositor** *is* the display server — there is no separate X
server brokering between clients and the compositor. Clients render their own
buffers and hand them to the compositor, which composites the final screen.

> [!key-insight]
> "Wayland" is a protocol, not a program. The thing you actually run is a
> compositor that speaks it — KWin (KDE), Mutter (GNOME), Hyprland, Sway,
> Gamescope. Behaviour and quirks are largely compositor-specific.

## Why it replaced X11

- **Security** — X11 lets any client read input and other windows' contents;
  Wayland isolates clients by design.
- **Simpler architecture** — no decades-old X server in the path; every frame is
  composited.
- **Per-output handling** — better mixed-DPI, fractional scaling, and per-monitor
  refresh.

## The trade-offs

- **Screen capture / remote control** go through PipeWire portals rather than
  reading the X root window — older tools needed updating.
- **Proprietary GPU drivers** (notably NVIDIA) were the slow adopters; this is
  why [[NVIDIA on Wayland]] is its own topic.
- **XWayland** runs legacy X11 apps inside a Wayland session for compatibility.

## Common building blocks

| Piece | Role |
|-------|------|
| Compositor | The Wayland server (KWin, Mutter, Hyprland, Sway) |
| XWayland | Compatibility layer for X11 apps |
| PipeWire + xdg-desktop-portal | Screen sharing, screenshots, screencast |
| libinput | Input device handling |

## Related

- [[NVIDIA on Wayland]] — the proprietary-driver pain points
- [[KDE Plasma]] — KWin is its Wayland compositor
