---
type: entity
title: ghostctl
created: 2026-06-21
updated: 2026-06-28
tags:
  - sysadmin
  - devops
  - rust
  - cli
  - linux
  - homelab
status: active
entity_type: repository
language: Rust
repo_status: active
purpose: All-in-one Rust sysadmin/DevOps/homelab CLI that replaces dozens of
  tools via interactive workflows and automation
related:
  - "[[Rust]]"
  - "[[Proxmox]]"
  - "[[VFIO GPU Passthrough]]"
  - "[[IOMMU and Device Groups]]"
  - "[[NVIDIA]]"
  - "[[Ansible]]"
  - "[[Terraform]]"
  - "[[CrowdSec]]"
  - "[[Self-Hosted Services]]"
  - "[[Ghost Ecosystem]]"
  - "[[GhostKellz]]"
---

# ghostctl

An all-in-one system-administration, DevOps, and homelab CLI that turns complex
Linux operations into interactive workflows and headless automation. It replaces
dozens of single-purpose tools behind one binary. Built in [[Rust]] (clap +
ratatui + tokio, with Lua scripting via mlua). Authored by [[GhostKellz]],
MIT-licensed.

## Language & stack

- **Language**: [[Rust]] — clap (CLI), ratatui (TUI), tokio (async).
- **Scripting**: Lua via mlua for user automation.
- **Form factor**: single binary with an interactive menu plus direct
  subcommands; global flags `--headless`, `--dry-run`, `--yes`, `--plain`,
  `--non-interactive`.

## Capability surface

### System & packages

- Arch: fix, clean, AUR, boot, health, optimize, mirrors, orphans.
- System update/status; [[Nix|NixOS]] rebuild/rollback; sysctl and kernel browser;
  shell/zsh; systemd service management.

### Virtualization & hardware

- [[Proxmox]] (`pve`): VM/CT lifecycle and bulk operations.
- [[VFIO GPU Passthrough]] (`vfio`): bind/unbind, single-GPU passthrough, ROM
  dump.
- [[IOMMU and Device Groups]] (`iommu`): group inspection and ACS handling.
- [[NVIDIA]]: driver, CUDA, DKMS, and Wayland setup.
- UEFI: Secure Boot key management.

### Storage & backup

- [[Btrfs]]: snapshots, scrub, balance, quota, plus Snapper.
- Backup: setup, schedule, verify; restore and chroot recovery.
- Storage backends: S3/MinIO, Azure, SFTP.

### Containers & DevOps

- [[Docker]]: install, registry, homelab stacks, cleanup.
- Cloud: AWS, Azure, GCP.

### Infrastructure as Code

- [[Ansible]], [[Terraform]], multi-cloud, and CI/CD templates.

### Security & keys

- SSH keygen, deploy, and GitHub/GitLab sync; GPG lifecycle.
- SSL via acme.sh; package signing (Pacman/DEB/RPM/PE).
- Security audits with OSV and RUSTSEC.

### Network & diagnostics

- DNS with DNSSEC, mesh, port scan, netcat; Wi-Fi via iwd; Bluetooth.

### Dev & tools

- Toolchains: [[Rust]], Zig, Go, Python, JS.
- Editors/shell: nvim/LazyVim, ghostty, starship; git/gitlab.
- AI: Ollama tuning; [[CrowdSec]]; monitoring with Prometheus and Wazuh.

Plus Nix rebuild/rollback, gaming optimization, and nginx management.

## Status

v0.12.x — production-ready and multi-distro (Arch, Ubuntu, Debian, Fedora, RHEL,
openSUSE, Proxmox). MIT-licensed.

> [!key-insight]
> ghostctl is the "Swiss-army knife" of the [[Ghost Ecosystem]]: rather than one
> narrow tool, it consolidates dozens of admin tasks behind interactive menus and
> scriptable automation.

## Relationships

- Built on [[Rust]]; part of the [[Ghost Ecosystem]].
- Heavy overlap with [[Proxmox]], [[VFIO GPU Passthrough]],
  [[IOMMU and Device Groups]], and [[NVIDIA]] passthrough workflows.
- Drives [[Ansible]]/[[Terraform]] IaC and [[Self-Hosted Services]] stacks.
