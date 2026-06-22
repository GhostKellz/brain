---
type: entity
title: "Ghostwarden"
created: 2026-06-21
updated: 2026-06-21
tags:
  - networking
  - firewall
  - nftables
  - linux
status: seed
entity_type: repository
language: Rust
repo_status: experimental
purpose: "Linux network guardian for nftables, bridges, VM networks and lab policy enforcement (ufw++)"
related:
  - "[[GhostWire]]"
  - "[[ghostctl]]"
  - "[[Ghost Ecosystem]]"
  - "[[GhostKellz]]"
---

# Ghostwarden

A Linux network guardian for nftables, bridges, VM networks, and lab policy
enforcement — described as "ufw++" for Linux bridges, nftables, container
networks, and VM labs. Marked experimental (can plan and apply real host
networking changes). By [[GhostKellz]]; part of the [[Ghost Ecosystem]].

Pairs with [[GhostWire]] (mesh VPN) on the networking side and overlaps with the
firewall-automation features in [[ghostctl]].
