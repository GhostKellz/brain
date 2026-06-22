---
type: entity
title: "ghostctl"
created: 2026-06-21
updated: 2026-06-21
tags:
  - sysadmin
  - devops
  - rust
  - cli
  - linux
status: seed
entity_type: repository
language: Rust
repo_status: active
purpose: "Professional system administration toolkit — an interactive all-in-one Linux/DevOps/homelab management suite in Rust"
related:
  - "[[Rust]]"
  - "[[Ghost Ecosystem]]"
  - "[[reaper]]"
  - "[[GhostKellz]]"
---

# ghostctl

A comprehensive system-administration platform that turns complex Linux
operations into interactive, menu-driven workflows. Built in [[Rust]] for
performance, aimed at power users, DevOps engineers, and homelabbers. Authored by
[[GhostKellz]], MIT-licensed.

## Language & stack

- **Language**: Rust (1.91+), OpenSSL.
- **Form factor**: single binary `ghostctl` with an interactive main menu plus
  direct subcommands (`ghostctl docker menu`, `ghostctl pve menu`, etc.).
- Source lives in the nested `ghostctl/` crate; ships man pages and packaging.

## Feature surface (breadth-first)

- **Infrastructure as Code** — Ansible playbook lifecycle, Terraform
  plan/apply/destroy, multi-cloud (AWS, Azure, GCP, DO, Hetzner, Linode).
- **Security & keys** — SSH key management with GitHub/GitLab integration, GPG
  lifecycle, SSL certs, security auditing.
- **Backups & data protection** — Btrfs snapshots/subvolumes, Snapper
  automation, Restic workflows, systemd-timer backup scripts.
- **Object storage** — MinIO cluster setup, erasure-code config, S3-compatible
  bucket operations.
- **Containers / DevOps** — Docker registry mirror, container cleanup, Compose/
  Swarm orchestration, GitHub template deployment.
- **Proxmox VE** — template lifecycle, storage migration, backup rotation/
  pruning, firewall automation, cluster ops, bulk VM/CT operations, 40+
  community helper scripts.
- **Arch Linux** — package management, AUR helper preference system
  (reaper/paru/yay), system fixes.
- **UEFI / Secure Boot** — OVMF VARS generation with Microsoft keys for Windows
  11 VMs, key enrollment via virt-fw-vars.
- **Dev environment** — Neovim health/plugins/LSP, ZSH + Powerlevel10k,
  tmux/terminal, Git workflows.

## Status

Active, production-tested per its own description, with a large CHANGELOG.

> [!key-insight]
> ghostctl is the "Swiss-army knife" of the [[Ghost Ecosystem]]: rather than one
> narrow tool, it consolidates dozens of admin tasks behind interactive menus.
> Its AUR-helper preference system ties into [[reaper]] and the broader Arch
> tooling.

## Relationships

- Built on [[Rust]]; part of the [[Ghost Ecosystem]].
- Integrates AUR helper preferences including [[reaper]]; complements the
  [[GhostView]] package GUI.
- Shares the homelab/Proxmox/admin focus that recurs across the ghost\* family.
