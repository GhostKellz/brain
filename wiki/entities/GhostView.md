---
type: entity
title: "GhostView"
created: 2026-06-21
updated: 2026-06-21
tags:
  - arch
  - gui
  - package-manager
  - rust
status: seed
entity_type: repository
language: Rust
repo_status: active
purpose: "Modular egui GUI for discovering and managing Arch Linux packages (Pacman, AUR, Flatpak)"
related:
  - "[[Rust]]"
  - "[[reaper]]"
  - "[[ghostctl]]"
  - "[[Ghost Ecosystem]]"
  - "[[GhostKellz]]"
---

# GhostView

A sleek, modular desktop GUI for discovering and managing packages across the
Arch Linux ecosystem, built with [[Rust]] and `egui`. By [[GhostKellz]].

Browses and manages Pacman (system/official), AUR and Chaotic-AUR, Flatpak
(Flathub + custom remotes), and KDE/GNOME apps. It sits in the [[Ghost
Ecosystem]] Arch tooling alongside the [[reaper]] AUR helper CLI and the
[[ghostctl]] admin toolkit.

> [!note]
> GhostView's own README still credits a "ghostbrew" backend, but that name has
> since been repurposed for the [[ghostbrew]] **sched-ext scheduler** — the two
> are unrelated. Treat GhostView as the standalone package GUI.

Built on [[Rust]].
