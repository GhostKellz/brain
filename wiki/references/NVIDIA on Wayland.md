---
type: reference
title: "NVIDIA on Wayland"
created: 2026-06-21
updated: 2026-06-21
tags:
  - nvidia
  - wayland
  - kde
  - linux
status: seed
related:
  - "[[Wayland]]"
  - "[[KDE Plasma]]"
  - "[[NVIDIA GSP Firmware]]"
---

# NVIDIA on Wayland

Running [[Wayland]] on proprietary NVIDIA drivers has historically been the rough
edge of Linux desktop. Modern RTX cards (40/50-series) on recent kernels +
nvidia-open drivers are largely fine; older cards (20/30-series) need more care.

> [!key-insight]
> In practice the newest cards are *less* work on Wayland, not more — 40/50-series
> on nvidia-open are noticeably less finicky than 20/30-series, which sometimes
> end up better served by [[VFIO GPU Passthrough]] + Looking Glass.

## Topics that bite

- Screen tearing and refresh-rate handling
- Hybrid iGPU/dGPU behaviour
- HDR / color-space support
- PipeWire-based screenshots/screencasts (e.g. Spectacle crashes)
- Session env vars and overrides
- Compositor-specific quirks (KDE/KWin, Hyprland, Sway, GNOME)

## The frozen-monitor workaround (KDE + Wayland + NVIDIA)

A known bug freezes the monitor while the session is actually still alive. Don't
reboot — bounce the VT:

```
Ctrl + Alt + F3     # switch to a TTY
Ctrl + Alt + F1     # switch back to the graphical session
```

The session isn't killed, just refreshed — far faster than logging out. The issue
appears reduced on kernels that run the GSP firmware at lower frequency (see
[[NVIDIA GSP Firmware]]).

## Driver/stack assumptions

These tweaks assume a modern stack:

- Recent kernel (7.0+ range)
- nvidia-open driver branch
- A Wayland compositor (KWin / Hyprland / Gamescope)

Wayland behaviour varies by compositor, so a fix for KWin may not apply to
Hyprland and vice versa.

> [!gap]
> Specific GPU models, kernel builds, and per-host display layouts are
> infrastructure detail and live in the private tier.

## Related

- [[Wayland]] — the display protocol
- [[NVIDIA GSP Firmware]] — disabling/throttling GSP for stability
- [[KDE Plasma]] — the desktop most affected here
