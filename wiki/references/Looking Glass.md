---
type: reference
title: "Looking Glass"
created: 2026-06-28
updated: 2026-06-28
aliases:
  - "LookingGlass"
  - "IVSHMEM"
  - "KVMFR"
tags:
  - looking-glass
  - vfio
  - virtualization
  - kvm
  - windows
  - gpu
status: developing
related:
  - "[[VFIO GPU Passthrough]]"
  - "[[IOMMU and Device Groups]]"
  - "[[Proxmox]]"
  - "[[Arch Linux Administration]]"
  - "[[CUDA Delivery Paths]]"
---

# Looking Glass

**Looking Glass** lets you use a GPU-passthrough Windows VM as if it were a native
window on your Linux desktop — no second monitor, no KVM switch, no dual-booting.
The Windows guest renders to a **real passed-through GPU** ([[VFIO GPU Passthrough|VFIO]]),
and Looking Glass copies each frame through a shared-memory region (**IVSHMEM**) so
a client on the Linux host displays it **uncompressed, with no video encode and no
network hop**. The result is a lossless, near-zero-latency Windows desktop sitting
right next to your Linux apps.

> [!key-insight]
> Dual-booting is dead. The old reason to keep a Windows partition was
> *full-speed GPU access* for Windows-only software (games, CUDA tools, Adobe,
> CAD). Looking Glass removes that reason: the guest drives a **real GPU at
> near-bare-metal speed**, and you see it in a window without rebooting out of
> Linux. You get both OSes, simultaneously, with full acceleration on one machine.

## My setup (Arch workstation)

A single Arch Linux daily driver (Ryzen 9950X3D · RTX 5090) acts as the **KVM
host**. The discrete **RTX 5090 is handed whole to a Windows guest** via VFIO; the
CPU's **Radeon iGPU keeps the `amdgpu` driver** and drives the Arch desktop — so
the host always has a display to render the Looking Glass client into.

```
Windows guest VM                          Arch Linux host · KVM + VFIO
┌────────────┐   render   ┌───────────┐  write   ┌─────────┐  read   ┌──────────┐   ┌───────────────┐
│ Windows    │──────────▶ │ LG host   │────────▶ │ IVSHMEM │───────▶ │ LG       │──▶│ Arch desktop  │
│ guest      │            │ app       │  frames  │ shared  │  frames │ client   │   │ lossless ·    │
│ q35 · OVMF │            │ (in guest)│          │ memory  │         │ (on host)│   │ ~no latency   │
│ passthru   │ ◀───────────────────────────────────────────────────────────────────│ input kbd/mouse│
│ GPU        │            └───────────┘          └─────────┘         └──────────┘   └───────────────┘
└────────────┘
```

The frames are **uncompressed and never leave RAM**; keyboard/mouse input is fed
back into the guest. 3D apps and CUDA workloads inside Windows run at effectively
bare-metal performance.

## How it works

### 1. GPU passthrough (the foundation)

Looking Glass is *not* a passthrough mechanism — it's a **display transport** that
sits on top of one. First you need a working [[VFIO GPU Passthrough|VFIO]] setup:

- Enable IOMMU on the host (`amd_iommu=on` / `intel_iommu=on`, `iommu=pt`).
- Bind the guest GPU (and its HDMI-audio function) to **`vfio-pci`** at boot so the
  host driver never claims it.
- Give the guest a **q35 + OVMF (UEFI)** machine type with PCIe passthrough.

> [!key-insight]
> The split only works cleanly when the GPUs land in **separate [[IOMMU and Device
> Groups|IOMMU groups]]**. Spec a motherboard with proper IOMMU layout up front and
> the discrete GPU isolates on its own — no **ACS-override** kernel patch needed.
> Fewer patches = cleaner upgrades and predictable passthrough. You also need a
> **second GPU (or iGPU)** to keep the host desktop alive; here the Radeon iGPU
> stays on `amdgpu`.

### 2. The IVSHMEM shared-memory bridge

Instead of an emulated display or a network stream, Looking Glass uses an
**IVSHMEM** (inter-VM shared memory) device — a chunk of RAM mapped into both the
guest and the host:

- The **LG host app** runs *inside Windows*, captures the guest framebuffer, and
  writes each frame into the IVSHMEM region (the KVMFR — kernel/KVM frame relay —
  protocol).
- The **LG client** runs *on the Linux host*, reads frames straight from that
  shared memory, and renders them into a normal window.

No H.264/NVENC encode, no decode, no TCP — just a memory copy. That's why it's
lossless and ~latency-free.

### 3. Sizing the IVSHMEM region

The shared-memory device must be large enough for the guest's resolution. Rule of
thumb: `width × height × 4 × 2`, rounded **up to the next power of two** (MiB).

| Resolution | Minimum IVSHMEM |
|------------|-----------------|
| 1920×1080  | 32 MB |
| 2560×1440  | 64 MB |
| 3840×2160 (4K) | 128 MB |

Undersize it and the client refuses to start.

## Host-side wiring (libvirt/QEMU)

The IVSHMEM device is exposed to the guest in the domain XML, backed by a
shared-memory file the host creates (commonly `/dev/shm/looking-glass`):

```xml
<!-- libvirt domain: add an ivshmem-plain device, size in MiB -->
<shmem name='looking-glass'>
  <model type='ivshmem-plain'/>
  <size unit='M'>128</size>
</shmem>
```

The backing file needs the right owner/permissions so the LG client can open it —
a tmpfiles rule keeps it across reboots:

```
# /etc/tmpfiles.d/10-looking-glass.conf
f /dev/shm/looking-glass 0660 <your-user> kvm -
```

Then on the host:

```bash
looking-glass-client            # opens the window, reads /dev/shm/looking-glass
# common flags:
looking-glass-client -F         # start fullscreen
looking-glass-client -s         # disable the (spice) audio/clipboard if unused
```

Inside Windows, install the **IVSHMEM driver** (the VirtIO/Red Hat ivshmem device)
and run the **Looking Glass host** (`looking-glass-host.exe`, set to autostart as a
service) so capture begins at login.

> [!note]
> Match the **host app and client versions** (same Looking Glass release, e.g. the
> B-series builds). A version mismatch between the Windows host app and the Linux
> client is the most common "black screen / won't connect" cause.

## Input, audio, clipboard

- **Input:** Looking Glass captures keyboard/mouse and feeds them back to the guest.
  A SPICE channel handles capture/release; a hotkey (default **ScrollLock**) toggles
  capture so input returns to the Linux host.
- **Audio:** route guest audio over a virtual sink (SPICE or, for lower latency,
  Scream over the IVSHMEM/virtio path).
- **Clipboard:** SPICE provides host↔guest clipboard sharing.

## Why this beats dual-booting

| | Dual-boot | Looking Glass |
|--|-----------|---------------|
| Switch OS | reboot | none — both run at once |
| Linux available while in Windows | no | yes |
| GPU performance in Windows | native | **near-native** (real GPU passthrough) |
| Windows-only apps / games / CUDA | yes | yes |
| Friction | high (reboot, lose state) | low (Alt-Tab) |
| Cost | one GPU | needs a 2nd GPU/iGPU for the host |

The only real tax is needing a **second display adapter** for the host. Given a
modern CPU with an iGPU (or a cheap secondary card), that's a non-issue — and the
payoff is never rebooting to run that one Windows-only tool again.

## Pros / trade-offs

**Pros**
- Lossless, low-latency Windows desktop in a Linux window; no encode/decode.
- Real GPU in the guest → near-bare-metal 3D/CUDA/game performance.
- Kills dual-booting; both OSes live simultaneously.

**Trade-offs**
- Requires working VFIO + clean IOMMU groups (hardware planning).
- Needs a **second GPU/iGPU** to keep the host desktop alive.
- Host-app/client version coupling; IVSHMEM must be sized to resolution.
- Windows-side driver + host-service setup adds moving parts.

## Related

- [[VFIO GPU Passthrough]] — the passthrough layer Looking Glass rides on
- [[IOMMU and Device Groups]] — why clean groups make this painless (no ACS patch)
- [[CUDA Delivery Paths]] — the same VFIO mechanism gives a VM full CUDA
- [[Proxmox]] — the cluster does the same passthrough for headless GPU VMs
- [[Arch Linux Administration]] — host-side `vfio-pci`, tmpfiles, libvirt wiring
