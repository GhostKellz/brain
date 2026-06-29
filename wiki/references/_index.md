---
type: meta
title: "References Index"
created: 2026-06-21
updated: 2026-06-28
tags:
  - meta
  - index
status: stable
related:
  - "[[Wiki Index]]"
---

# References

Runbooks and how-to procedures.

## Linux — distro admin
- [[Linux Administration]] (hub) · [[Arch Linux Administration]] ·
  [[Debian and Ubuntu Administration]] · [[Fedora Administration]] ·
  [[RHEL, Rocky and Alma Administration]] · [[openSUSE Administration]]

## Linux — packaging, updates & tuning
- [[Pacman Hooks]] · [[PKGBUILD Templates]] · [[PKGBUILD Auditing]] ·
  [[Modprobe Options]] · [[Ubuntu Unattended Upgrades]] ·
  [[Sysctl Performance Tuning]] · [[Disk IO Scheduler]] ·
  [[systemd]] · [[Initramfs FSCK Recovery]]

## Filesystems
- [[Btrfs]] (subvolume layout, restore, troubleshooting) · [[ZFS]] ·
  [[ReFS]] (Windows CoW + block cloning; the Veeam repo filesystem)

## Backup
- [[Restic Backup]] · [[Cloud Backup Storage]] ·
  [[Snapper]] (Btrfs local-rollback: snap-pac + timeline) ·
  [[Veeam|Veeam Backup & Replication]] (image-level VM backup, SOBR, offsite to cloud) ·
  [[Comet Backup]] (file-level + image/Hyper-V; client-side dedup to your own Wasabi) ·
  [[Proxmox Backup Server]] (dedup VM/CT backup + S3 object-storage datastores)

## Networking
- [[Networking Reference]] · [[Linux Networking]] (bridges/bonds/VLANs/VXLAN) ·
  [[nftables Firewall]] · [[FortiGate Administration]] ·
  [[UniFi Controller]] · [[Tailscale]] ·
  [[Headscale]] · [[Cloudflare]]

## acme.sh & TLS certificates
- [[acme.sh - DNS-01 Certificates|acme.sh]] — DNS-01 (Cloudflare + Azure DNS),
  wildcards, Let's Encrypt issuance & auto-renew
- [[Let's Encrypt - Certbot|Let's Encrypt / Certbot]] — HTTP-01 / nginx plugin

## Code signing
- [[Azure Key Vault Code Signing]] — HSM-backed Authenticode signing
  (AzureSignTool) + ScreenConnect / ConnectWise Control agents

## Web & hosting
- [[Nginx Reference]] · [[Self-Hosted Services]] · [[Docker]] ·
  [[Portainer]] (archived) · [[HestiaCP]]

## GPU & Wayland
- [[NVIDIA]] · [[NVIDIA Container Runtime Troubleshooting]] ·
  [[Looking Glass]] (VFIO passthrough Windows VM in a lossless host window)

## AI
- [[Ollama Service Configuration]] · [[hermes-agent]] (self-improving,
  model-agnostic autonomous agent harness)

## Hudu
- [[Hudu]] — wiring Claude Code + Codex into Hudu via its native MCP to read/write
  client & internal KBs (OAuth, two-tier knowledge model, leak guard)

## Remote access / RMM
- [[ScreenConnect]] (ConnectWise Control) — self-hosted remote support; public
  relay + tailnet-only admin portal, Toolbox & clipboard passthrough

## Cloud & Microsoft
- [[Microsoft 365 Administration]] · [[Exchange Online Administration]] ·
  [[Active Directory Administration]] · [[Windows Administration]] ·
  [[Windows Activation]] · [[Hyper-V Administration]] · [[Proxmox]] ·
  [[macOS Virtualization]] · [[Pax8]] (MSP cloud marketplace/distributor)

## Observability
- [[Uptime Kuma]] (uptime/synthetic + 90+ notifiers) · [[Prometheus Monitoring]] ·
  [[Grafana]]

## Communications / ChatOps
- [[Discord]] (webhooks, bots, alert hub, vanity invite) · [[SMTP2GO]] (outbound
  SMTP relay for app/alert mail)

## Security
- [[Linux Server Hardening]] (the hardening bible) · [[Endpoint Security Tooling]] ·
  [[Huntress]] (managed EDR/ITDR/SOC) · [[Avanan]] (API email security) ·
  [[Email DNS Security]] · [[Wazuh]] · [[CrowdSec]]

## Infrastructure as code
- [[Terraform]] (provision: VMs/DNS/cloud) · [[Ansible]] (configure: agentless
  over SSH) — Terraform builds, Ansible configures; cloud-init bootstraps on
  first boot (see [[Proxmox]])

## Dev workflow
- [[Cargo Workflow]] · [[CICD Workflows]] · [[tmux Cheatsheet]] · [[Neovim Cheatsheet]]

## Phonetic alphabet
- [[NATO Phonetic Alphabet]] — spelling letters/digits aloud over phone or radio
