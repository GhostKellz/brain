---
type: reference
title: "macOS Virtualization"
created: 2026-06-21
updated: 2026-06-21
tags:
  - macos
  - virtualization
  - qemu
status: developing
related:
  - "[[Proxmox Administration]]"
  - "[[Windows Activation]]"
---

> [!key-insight] Generalized from field notes; host/client-specific values are placeholders.

Notes for running macOS as a guest under QEMU/KVM-based hypervisors.

## OSK Board Key

macOS guests check for an Apple SMC board key (the "OSK"). This well-known string is widely published and is required by QEMU-based macOS setups to satisfy the SMC check:

```
ourhardworkbythesewordsguardedpleasedontsteal(c)AppleComputerInc
```

> [!note] This OSK is a long-public, widely-documented constant — not a license key. Running macOS in a VM still requires you to comply with Apple's licensing (generally on Apple hardware).

## QEMU/KVM Host Packages (Linux)

```bash
sudo apt install -y qemu qemu-kvm libvirt-daemon libvirt-clients bridge-utils virt-manager
sudo systemctl status libvirtd
```

## Notes

- For running guests on Proxmox specifically, see [[Proxmox Administration]].
- Windows guest activation is unrelated and uses KMS/GVLK; see [[Windows Activation]].
