---
type: entity
title: ghost-kde
created: 2026-06-28
updated: 2026-06-28
tags:
  - kde
  - plasma
  - theming
  - rust
  - tokyo-night
  - ricing
status: developing
entity_type: repository
language: Rust
repo_status: experimental
purpose: Complete KDE Plasma 6 theming toolkit in Rust applying a Tokyo Night
  palette across the desktop, login, boot, browser, terminal, and shell
related:
  - "[[KDE Plasma]]"
  - "[[Rust]]"
  - "[[Tiling Window Management]]"
  - "[[Ghost Ecosystem]]"
  - "[[GhostKellz]]"
---

# ghost-kde

A complete [[KDE Plasma]] 6 theming toolkit written in [[Rust]], applying a
Tokyo Night palette across the desktop, login (SDDM), boot (Plymouth/GRUB),
browser, terminal, and shell. Authored by [[GhostKellz]] (CK Technology LLC),
MIT-licensed.

## Language & stack

- **Language**: Rust.
- **CLI**: clap 4.6.
- **TUI**: ratatui (interactive installer).
- **Errors**: color-eyre.
- **Config**: TOML.

## Capabilities

- **20 theme components**: Aurorae window decorations, Plasma panels/widgets,
  Kvantum (Qt), KDE color schemes, SDDM, Plymouth, GRUB, GTK3/4, Konsole,
  Ghostty, icon themes (Papirus/Tela/WhiteSur), Firefox userChrome, Bibata
  cursors, wallpapers, Yakuake, and KWin tiling configs (Krohnkite/Polonium/
  KZones).
- **3 variants**: Night, Storm, and Moon.
- **Translucency**: opt-in glass/translucency via `--opacity` (default 88%).
- **Shell integration**: Zsh with Powerlevel10k and Starship, plus LS_COLORS.
- **Backup / restore**: `.tar.gz` snapshots before applying changes.
- **Installation**: interactive TUI plus modular per-component install.
- **Fonts & presets**: JetBrains Mono Nerd Font and panel presets.

## Relationships

Applies KWin tiling layouts via Krohnkite, Polonium, and KZones — see
[[Tiling Window Management]]. Built on [[Rust]]; part of the [[Ghost Ecosystem]]
desktop tooling.

## Status

v0.1.0, early. Available on the AUR; targets Plasma 6.6. MIT-licensed.
