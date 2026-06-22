---
type: concept
title: "CachyOS and TKG Kernels"
created: 2026-06-22
updated: 2026-06-22
tags:
  - kernel
  - cachyos
  - tkg
  - bore
  - eevdf
  - gaming
  - performance
  - amd
  - zen5
  - arch
status: developing
related:
  - "[[eBPF and Linux Schedulers]]"
  - "[[ghostbrew]]"
  - "[[Linux and Systems]]"
  - "[[Arch Linux Administration]]"
  - "[[GhostKellz]]"
---

> [!key-insight] CachyOS and TKG are both **custom kernel recipes** built on top
> of mainline Linux. They ship the same family of latency/throughput patches
> (BORE, full tickless, LTO, `-march` CPU tuning) and both enable
> `CONFIG_SCHED_CLASS_EXT`, so you can also load a [[ghostbrew|sched-ext]]
> scheduler on top. The difference is packaging, not philosophy.

Two Arch-friendly kernel builds used here for a low-latency, gaming-tuned
desktop on an AMD Zen 5 (Ryzen 9000-series / X3D) machine. Both are compiled
locally so the CPU-specific (`znver5`) and scheduler options are baked in.

| Priority | Kernel | Scheduler | Source |
|----------|--------|-----------|--------|
| **Primary** | `linux-cachyos-lto` | EEVDF + BORE | [CachyOS/linux-cachyos](https://github.com/CachyOS/linux-cachyos) |
| **Fallback** | `linux-ghost` (TKG) | BORE | [Frogging-Family/linux-tkg](https://github.com/Frogging-Family/linux-tkg) |
| **Backup** | `linux-zen` | EEVDF | Arch repos |

Keeping a stock `linux-zen` (or `linux-lts`) installed as a third boot entry is
the safety net: if a custom build regresses, you still have a known-good kernel
in the boot menu.

## linux-cachyos-lto (primary)

The CachyOS PKGBUILD exposes build knobs as `_variables`. The tuned set:

```bash
_cpusched="cachyos"      # EEVDF + BORE interactivity patches
_use_llvm_lto="full"     # full Clang/ThinLTO link-time optimization
_processor_opt="zen5"    # -march=znver5 codegen for Ryzen 9000 / X3D
_HZ_ticks="1000"         # 1000 Hz timer
_tickrate="full"         # full tickless (nohz_full) — fewer timer wakeups
_preempt="full"          # PREEMPT — lowest scheduling latency
_hugepage="always"       # transparent hugepages always on
_cc_harder="yes"         # -O3 + harder optimization flags
```

For brand-new silicon, Zen 5 (`CONFIG_MZEN5`) may not yet exist in the kernel's
Kconfig. A small patch adds the `MZEN5` option and a matching `ZEN5` case in the
PKGBUILD's CPU switch so the build emits true `znver5` code instead of a generic
fallback. This mirrors CachyOS's own pre-built `znver5` repository binaries —
building locally just lets you combine it with LTO + your own config fragment.

## linux-ghost / TKG (fallback)

Built via the Frogging-Family **linux-tkg** generator (`customization.cfg`).
Equivalent tuning, BORE as the baseline scheduler, packaged under a custom name:

```bash
_cpusched="bore"            # BORE (Burst-Oriented Response Enhancer)
_compiler="llvm"            # Clang/LLVM
_lto_mode="full"            # full LTO
_processor_opt="znver5"     # -march=znver5 (no Kconfig patch needed)
_timer_freq="1000"          # 1000 Hz
_tickless="1"               # full tickless
_default_cpu_gov="performance"
_tcp_cong_alg="bbr"         # BBR congestion control
_zenify="true"              # Zen/Liquorix interactivity patches
_glitched_base="true"       # community patch set
_custom_pkgbase="linux-ghost"
```

TKG handles `znver5` purely through `-march`, so no Kconfig patch is required —
a simpler path when you don't need CachyOS's extra scheduler choices.

## Why these patches help

- **BORE** rebalances EEVDF/CFS toward interactive, bursty tasks (input, audio,
  game threads) — see [[eBPF and Linux Schedulers]] for where BORE sits relative
  to sched-ext.
- **1000 Hz + full tickless + `PREEMPT`** trade a little throughput for much
  lower and more consistent scheduling latency — what you want for gaming and
  responsive desktops.
- **Full LTO + `-march=znver5` + `-O3`** let the compiler specialize for the
  exact CPU and inline across translation units; measurable on tight kernel hot
  paths.
- **THP always + `performance` governor** reduce TLB misses and clock-ramp
  stalls under load.
- **BBR** improves throughput/latency on lossy or high-BDP links.

All of these enable `CONFIG_SCHED_CLASS_EXT=y`, so [[ghostbrew]] (or any
`scx_*` scheduler) can be hot-loaded on top of the BORE/EEVDF baseline.

## Recommended boot parameters

Passed via systemd-boot entries (`/boot/loader/entries/`) or GRUB. Why each one:

```bash
# memory / latency
zswap.enabled=0                 # prefer zram instead of zswap
nowatchdog                      # drop the NMI/soft watchdog → less jitter
quiet loglevel=3                # quieter, faster boot

# CPU / power
amd_pstate=passive              # let the governor drive P-states directly
processor.max_cstate=5          # cap deep C-states → lower wake latency

# NVIDIA (for an NVIDIA + Wayland rig)
nvidia_drm.modeset=1                            # required for Wayland/KMS
nvidia.NVreg_PreserveVideoMemoryAllocations=1   # clean suspend/resume
nvidia.NVreg_UsePageAttributeTable=1            # PAT → better VRAM throughput
nvidia.NVreg_EnableGpuFirmware=0                # disable GSP firmware (stability)
usbcore.autosuspend=-1                          # no USB autosuspend (audio/HID)
```

The NVIDIA flags only apply on an NVIDIA box; on AMD/Intel graphics drop them.
See [[NVIDIA on Wayland]] and [[NVIDIA GSP Firmware]] for the GPU side.

## Building

Both are normal `makepkg` PKGBUILDs (run from each kernel's build dir):

```bash
# CachyOS
makepkg -si        # installs as linux-cachyos-lto

# TKG (or use its ./install.sh wizard)
makepkg -si        # installs as linux-ghost
```

Using `modprobed-db` (TKG `_modprobeddb="true"`) trims the module set to only
what your hardware loads, cutting build time significantly.

## Related

- [[eBPF and Linux Schedulers]] — sched-ext / `scx`, and where BORE/EEVDF fit.
- [[ghostbrew]] — the custom `scx` scheduler loaded on top of these kernels.
- [[Arch Linux Administration]] · [[Linux and Systems]] — the wider setup.
