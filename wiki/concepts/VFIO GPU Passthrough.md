---
type: concept
title: "VFIO GPU Passthrough"
created: 2026-06-21
updated: 2026-06-21
tags:
  - virtualization
  - vfio
  - kvm
  - nvidia
status: seed
related:
  - "[[CUDA Delivery Paths]]"
  - "[[NVIDIA Container Toolkit]]"
  - "[[IOMMU and Device Groups]]"
---

# VFIO GPU Passthrough

**VFIO passthrough** hands a physical GPU directly to a virtual machine. The host
binds the card to the `vfio-pci` stub driver (so the host never uses it), and a
KVM/QEMU guest installs its own GPU driver as though the card were physically its
own. This gives **near-native performance with hard isolation**.

> [!key-insight]
> The whole point is exclusivity: a passed-through GPU belongs entirely to one
> guest. If you instead want *many* workloads sharing one card, that's the
> [[NVIDIA Container Toolkit]] path, not VFIO. See [[CUDA Delivery Paths]].

## How it works

```
Host kernel
  ├─ IOMMU (VT-d / AMD-Vi) groups devices for safe isolation
  ├─ GPU bound to vfio-pci (NOT nouveau/nvidia on the host)
  └─ KVM/QEMU guest (q35 + OVMF/UEFI) ← GPU appears as a real PCI device
        └─ guest installs its own nvidia driver
```

Requirements:

- **IOMMU** enabled in firmware and kernel (`intel_iommu=on` / `amd_iommu=on`).
- The GPU sits in a clean **IOMMU group** (see [[IOMMU and Device Groups]]).
- Host blacklists the GPU's normal driver and binds `vfio-pci` instead.
- Guest uses **q35** machine type with **OVMF** (UEFI) firmware.

## Use cases

- **Windows gaming VM** on a Linux host.
- **A dedicated inference/compute VM** that owns a whole GPU.
- Running another distro with its own GPU stack for testing.

## Looking Glass — using a passthrough VM as a desktop

A passthrough GPU normally outputs to its own physical display. **Looking Glass**
copies the guest's framebuffer back to the host over shared memory, so you can use
the VM in a window on your host desktop at low latency — no second monitor or KVM
switch.

## Stacking with containers

A common cluster pattern: pass a GPU into a Linux VM via VFIO, then run the
[[NVIDIA Container Toolkit]] *inside* that VM so many containers fan out over the
single passed-through card — isolation at the VM boundary, sharing within it.

> [!gap]
> Specific host PCI IDs, IOMMU groupings, and which cards are passed to which VMs
> are infrastructure detail and live in the private tier.

## Related

- [[CUDA Delivery Paths]] — VFIO vs native vs container
- [[IOMMU and Device Groups]] — the isolation layer passthrough depends on
- [[NVIDIA Container Toolkit]] — sharing a card instead of isolating it
