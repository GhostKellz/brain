---
type: concept
title: "Ghostty Terminal"
created: 2026-06-28
updated: 2026-06-28
tags:
  - ghostty
  - terminal
  - gpu
  - wayland
  - linux
status: seed
related:
  - "[[Zsh and Powerlevel10k]]"
  - "[[tmux Cheatsheet]]"
  - "[[Wayland]]"
  - "[[NVIDIA]]"
---

# Ghostty Terminal

**Ghostty** is a fast, GPU-accelerated terminal emulator written in [[Zig]] by
Mitchell Hashimoto. It targets native platform integration (real GTK on Linux,
native on macOS) with a small, opinionated config file and sane defaults — you
only set what you want to change.

> [!key-insight]
> Ghostty's pitch is "fast and native with zero ceremony." Unlike terminals that
> need a sprawling config to be usable, it ships good defaults and a flat
> `key = value` file — so a daily-driver config is often a dozen lines: font,
> size, and a color palette. GPU rendering + a low-overhead protocol make it feel
> instant even under heavy scrollback.

## Why GPU-accelerated terminals

Rendering glyphs on the GPU (vs the CPU drawing each cell) keeps the terminal
smooth when output floods in — `cat` of a large file, fast log tails, full-screen
TUIs. On a [[Wayland]] + [[NVIDIA]] setup this pairs with the GPU stack already in
play; Ghostty works on Wayland natively and falls back to XWayland if needed.

## Configuration

The config lives at `~/.config/ghostty/config` — flat `key = value` pairs, `#`
for comments (no inline comments — a `#` after a value becomes part of the value).

```ini
# font
font-family = CaskaydiaCove NFM SemiBold
font-size   = 15

# a "hacker blue" palette (TokyoNight-Moon-ish blend)
foreground   = #57c7ff
background    = #0d1117
cursor-color  = #57c7ff
```

Handy facts:

- `ghostty +show-config --default --docs` dumps every option with inline docs.
- Live reload: `Ctrl + Shift + ,` (Linux) re-reads the config; some options only
  apply to new windows / need a restart.
- Empty value (`key =`) resets a key to its default.
- A **Nerd Font** (patched, e.g. CaskaydiaCove/JetBrainsMono NFM) is what makes
  [[Zsh and Powerlevel10k|Powerlevel10k]] glyphs and dev-tool icons render.

## Where it sits in the stack

Ghostty is just the surface; the experience comes from what runs inside it:

| Layer | Piece |
|-------|-------|
| Terminal | **Ghostty** (GPU render, font, palette) |
| Multiplexer | [[tmux Cheatsheet|tmux]] — panes/windows/sessions |
| Shell + prompt | [[Zsh and Powerlevel10k|Zsh + Powerlevel10k]] |
| Colors | `vivid`-generated `LS_COLORS` (matching theme) |
| Editor | [[Neovim Cheatsheet|Neovim]] |

A cohesive look comes from sharing one palette across Ghostty, the prompt, `vivid`
`LS_COLORS`, tmux, and the Neovim theme.

## Related

- [[Zsh and Powerlevel10k]] — the shell + prompt that runs inside it
- [[tmux Cheatsheet]] — multiplexer for panes/sessions
- [[Wayland]] · [[NVIDIA]] — the display/GPU stack it renders on
- [[Zig]] — the language Ghostty is written in
