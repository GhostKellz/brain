---
type: concept
title: "NVIDIA GSP Firmware"
created: 2026-06-21
updated: 2026-06-21
tags:
  - nvidia
  - linux
  - firmware
  - troubleshooting
status: seed
related:
  - "[[NVIDIA]]"
  - "[[Modprobe Options]]"
---

# NVIDIA GSP Firmware

**GSP** (GPU System Processor) is a RISC-V microcontroller on modern NVIDIA GPUs.
Recent drivers — especially the **open** kernel modules — offload much of the
driver's work to firmware running on this processor. When that firmware
misbehaves, it can manifest as instability, hangs, or frozen displays.

> [!key-insight]
> A class of "random hang / frozen monitor" issues on NVIDIA + Linux trace back
> to GSP firmware behaviour. Disabling or throttling GSP is a common stability
> workaround on affected setups.

## The workaround

GSP can be disabled via a kernel module option, typically in
`/etc/modprobe.d/`:

```
# /etc/modprobe.d/nvidia.conf  (example — verify the option for your driver)
options nvidia NVreg_EnableGpuFirmware=0
```

Apply by regenerating the initramfs (so the option is present at module load) and
rebooting:

```bash
sudo mkinitcpio -P
```

See [[Modprobe Options]] for how module parameters are set and made persistent.

## Caveats

- The **open** driver may *require* GSP for some cards/features — disabling it is
  not universally safe; test on your hardware.
- Some users see the frozen-monitor bug *reduced* (not eliminated) on kernels
  that run GSP at lower frequency rather than disabling it outright.
- Exact option names have shifted across driver generations — confirm against the
  driver version in use.

> [!gap]
> Whether to disable GSP, and the exact option, depend on the specific GPU and
> driver branch on a given host (private-tier detail).

## Related

- [[NVIDIA#NVIDIA on Wayland|NVIDIA on Wayland]] — where the frozen-monitor symptom shows up
- [[Modprobe Options]] — setting kernel module parameters
