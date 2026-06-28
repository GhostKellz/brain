---
type: entity
title: "KDE Plasma"
created: 2026-06-21
updated: 2026-06-28
tags:
  - kde
  - kwin
  - desktop-environment
  - wayland
  - tiling
  - hdr
  - linux
status: developing
related:
  - "[[Wayland]]"
  - "[[NVIDIA]]"
  - "[[Tiling Window Management]]"
---

# KDE Plasma

**KDE Plasma** is a feature-rich, highly configurable Linux desktop environment.
Its compositor/window manager is **KWin**, which runs as both an X11 and a
[[Wayland]] compositor. Plasma 6 is Wayland-first, and it's the desktop where the
[[NVIDIA#NVIDIA on Wayland|NVIDIA-on-Wayland]] interplay (HDR, VRR, explicit sync)
matters most.

> [!key-insight]
> Plasma's strength is that almost everything is a config file under
> `~/.config` (`kwinrc`, `kglobalshortcutsrc`, `kdeglobals`, …). That means a
> Plasma setup — theme, shortcuts, tiling, HDR — is fully reproducible from
> dotfiles, no clicking through System Settings on a fresh install.

## Key components

| Component | Role |
|-----------|------|
| **KWin** | Window manager + compositor (X11 and Wayland) |
| **Plasma Shell** | Panels, widgets, system tray, desktop |
| **PowerDevil** | Power management (lid, sleep, brightness) |
| **Spectacle** | Screenshot tool (uses PipeWire portals on Wayland) |
| **KWin scripts** | Extend window behaviour (e.g. tiling via Krohnkite) |
| **System Settings** | The GUI front-end to the `*rc` config files |

## Config files that matter

Everything lives under `~/.config`; the load-bearing ones:

| File | Controls |
|------|----------|
| `kwinrc` | Compositor: effects, tiling, night color, decoration theme, plugins |
| `kglobalshortcutsrc` | Every global keybinding (KWin, Plasma, Krohnkite) |
| `kdeglobals` | Colors, fonts, widget style, icon theme |
| `plasma-org.kde.plasma.desktop-appletsrc` | Panels, widgets, layout |
| `kscreen*` | Per-output resolution / refresh / HDR / scaling |

Keep these in version control and a fresh machine reproduces the whole desktop.

## Theming

Plasma theming is layered — each layer is independent, which is why a "look" is
really four or five separate settings:

| Layer | What it skins | Set in |
|-------|---------------|--------|
| **Global Theme** | Bundles the layers below into one preset | *Appearance → Global Theme* |
| **Plasma Style** | Panels, widgets, system tray | *Appearance → Plasma Style* |
| **Application Style** | Qt widget style (Breeze, Kvantum) | *Appearance → Application Style* |
| **Window Decorations** | Titlebars / borders (Aurorae SVG themes) | *Appearance → Window Decorations* |
| **Colors / Icons / Cursors** | Palette, icon set, cursor theme | *Appearance → …* |

A common cohesive setup is an **Aurorae** SVG window-decoration theme (e.g.
`Sweet-ambar-blue`) paired with a matching Plasma style and a dark color scheme.
Aurorae decorations are just SVG + a small config, so they're easy to drop in and
track in dotfiles.

For GTK apps to match Qt, install a matching GTK theme and point GTK at it
(Plasma's *Application Style → Configure GTK Application Style* writes this for
you). Kvantum (`qt5ct`/`qt6ct` + Kvantum engine) gives finer control over Qt
theming than Breeze alone.

```ini
# kwinrc — window decoration (Aurorae SVG theme)
[org.kde.kdecoration2]
theme=__aurorae__svg__Sweet-ambar-blue
```

## HDR & VRR on Plasma 6

KWin on Plasma 6 has working **HDR** and **adaptive sync (VRR)** on the
nvidia-open driver — this is the big reason to be on Plasma 6 + Wayland for an
OLED gaming monitor.

### Enabling HDR for an OLED monitor

1. *System Settings → Display & Monitor →* select the display *→* toggle **HDR**.
2. Set **SDR brightness** (how bright non-HDR content renders inside the HDR
   container) — OLED usually wants this fairly low to avoid washed-out SDR.
3. Optionally enable **wide color gamut (WCG)** for the extra coverage OLED panels
   provide.
4. Apps/games opt into HDR explicitly:
   - Gamescope: `gamescope --hdr-enabled …`
   - Proton: `PROTON_ENABLE_HDR=1`
   - Native Vulkan titles negotiate it via the swapchain.

> [!key-insight]
> OLED + HDR on Plasma is a *per-display container*, not a global filter. SDR apps
> keep rendering in SDR and get mapped into the HDR space at the brightness you
> set — so getting "SDR brightness" right matters more than any single toggle.
> Burn-in mitigation (panel-side pixel-shift / logo dimming, plus a short DPMS /
> screen-off timeout) is worth keeping on for a desktop OLED.

### VRR / adaptive sync

*System Settings → Display & Monitor → Adaptive Sync →* **Automatic** (VRR only
when content changes, good for desktop) or **Always**. Pair with per-game
`__GL_VRR_ALLOWED=1` on NVIDIA. Night-color/color management live in the same
panel.

```ini
# kwinrc — constant night color (no time-based shifting), neutral temp
[NightColor]
Mode=Constant
NightTemperature=6500
```

GPU-side launch flags, explicit sync and the broader driver story are in
[[NVIDIA]].

## Krohnkite — dynamic tiling on KWin

Plasma floats by default. **Krohnkite** is a KWin script that adds i3/dwm-style
dynamic tiling with vim-direction bindings, while leaving Plasma's panels and
window management intact. Enable it as a plugin, then it manages layouts per
desktop/activity.

```ini
# kwinrc — enable the Krohnkite script + small tiling padding
[Plugins]
krohnkiteEnabled=true

[Tiling]
padding=4
```

### Layouts

Krohnkite ships several layouts you cycle through: **Tile** (master + stack),
**Monocle** (one window fills the screen), **Three-Column**, **Spiral**,
**Quarter**, **Stacked**, **Columns**, **Spread**, **Stair**, **Floating**.
Tile and Monocle cover most day-to-day use.

### Bindings (`Meta` = Super)

| Action | Keys |
|--------|------|
| Focus left / down / up / right | `Meta + H / J / K / L` |
| Move window | `Meta + Shift + H / J / K / L` |
| Grow width / height | `Meta + Ctrl + L` / `Meta + Ctrl + J` |
| Shrink width / height | `Meta + Ctrl + H` / `Meta + Ctrl + K` |
| Set window as master | `Meta + Return` |
| Increase master count | `Meta + I` |
| Monocle layout | `Meta + M` |
| Next / previous layout | `Meta + \` / `Meta + ⏷` (`Meta+Shift+\`) |
| Toggle float (one window) | `Meta + F` |
| Toggle float all (panic) | `Meta + Shift + F` |

These intentionally mirror [[tmux Cheatsheet|tmux]] and vim splits — one
`h/j/k/l` reflex across GUI windows, terminal panes, and editor splits.

### Floating the right windows

Krohnkite tiles everything *except* an ignore list. Games, Steam, and Discord are
typically excluded so they float/fullscreen normally instead of being forced into
a tile, and utility/dialog/splash windows auto-float:

```ini
# Krohnkite config — float chat/games + utility windows, gaps, per-desktop layouts
ignoreClass=...,steam,steam_app,discord
floatUtility=true
layoutPerDesktop=true
layoutPerActivity=true
```

> [!key-insight] The "everything stopped tiling" panic sequence
> When a layout misbehaves (a window stuck fullscreen, nothing tiling), three
> shortcuts fix almost every case:
> 1. `Meta + Shift + F` — toggle float-all back off
> 2. `Meta + \` — cycle layouts back to Tile/Columns
> 3. `Meta + F` — float/unfloat the offending window
>
> A single window eating the whole screen is usually just the **Monocle** layout —
> `Meta + \` (or `Meta + M`) cycles out of it.

Full conceptual background and the cross-tool `h/j/k/l` model live in
[[Tiling Window Management]].

## Useful default KWin shortcuts

Beyond Krohnkite, the stock KWin/Plasma bindings still apply:

| Action | Keys |
|--------|------|
| Quick-tile left / right | `Meta + Left` / `Meta + Right` |
| Quick-tile top / bottom | `Meta + Up` / `Meta + Down` |
| Maximize / minimize | `Meta + PgUp` / `Meta + PgDown` |
| Overview | `Meta + W` |
| Grid view | `Meta + G` |
| Edit tiles (KWin native tiling) | `Meta + T` |
| Switch desktop | `Meta + Ctrl + ←/→/↑/↓` |
| Move window to next/prev screen | `Meta + Shift + →` / `←` |
| Lock session | `Meta + L` |
| Clipboard history at cursor | `Meta + V` |
| Close window | `Alt + F4` |
| Kill window (force) | `Meta + Ctrl + Esc` |

> [!note] `Meta + L` collision
> Plasma's default **Lock Session** is `Meta + L`, which clashes with Krohnkite's
> "focus right". When running Krohnkite, one of the two gets rebound (lock → e.g.
> `Meta + Escape`, or focus-right stays on `Meta + L` and lock moves). Check
> `kglobalshortcutsrc` if a binding seems "eaten".

## Wayland on KDE

KWin is a mature Wayland compositor, but on NVIDIA it inherits proprietary-driver
quirks — notably a frozen-monitor bug worked around by switching to a TTY and back
(`Ctrl+Alt+F3` then `Ctrl+Alt+F1`) rather than rebooting. Full detail in
[[NVIDIA]].

## Related

- [[Wayland]] — KWin is its compositor
- [[NVIDIA]] — driver pain points, HDR/VRR, explicit sync on Plasma
- [[Tiling Window Management]] — the Krohnkite workflow and cross-tool model
