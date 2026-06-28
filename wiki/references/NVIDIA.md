---
type: reference
title: "NVIDIA"
created: 2026-06-28
updated: 2026-06-28
aliases:
  - nvidia-open
  - nvidia-smi
  - NVIDIA on Wayland
tags:
  - nvidia
  - linux
  - wayland
  - gaming
  - cuda
  - drivers
status: developing
related:
  - "[[NVIDIA GSP Firmware]]"
  - "[[NVIDIA Container Toolkit]]"
  - "[[Modprobe Options]]"
  - "[[Wayland]]"
  - "[[KDE Plasma]]"
  - "[[VFIO GPU Passthrough]]"
  - "[[nvcontrol]]"
---

# NVIDIA

The grand reference for running NVIDIA on Linux: the **open** kernel modules,
driver tuning via modprobe, Wayland, gaming/Gamescope, digital vibrance, and
AI/CUDA. The container runtime is its own concern — see
[[NVIDIA Container Toolkit]].

> [!key-insight]
> Modern NVIDIA on Linux is a different world than its reputation. On the
> **nvidia-open** branch (610+) with a recent kernel, 40/50-series cards are
> *less* work on Wayland than the old 20/30-series ever were. Most remaining pain
> is GSP firmware (see [[NVIDIA GSP Firmware]]) and compositor-specific sync —
> both now largely solved by explicit sync + the right modprobe options.

## Driver branches

| Branch | What it is |
|---|---|
| **nvidia-open** | open-source kernel modules (MIT/GPL), NVIDIA's strategic default for Turing+ (RTX 20-series and newer). The 610 branch is solid on recent kernels |
| **nvidia (proprietary)** | the classic closed kernel modules; being wound down in favour of open for supported cards |
| **nouveau / NVK** | community driver + Vulkan; fine for desktop on older cards, not for CUDA or heavy gaming |

> [!note]
> "Open" here means the **kernel modules** are open — the userspace (CUDA,
> Vulkan ICD, NVENC) is still proprietary. Turing and later have on-GPU GSP
> firmware that the open modules lean on heavily.

## Building nvidia-open from source

For bleeding-edge kernels (7.x) the open modules can be built straight from
NVIDIA's `open-gpu-kernel-modules` tree and managed with **DKMS** so they rebuild
automatically on every kernel bump:

```bash
# clone the matching driver branch (e.g. 610) and build/install the modules
make modules -j"$(nproc)"
sudo make modules_install
```

A minimal `dkms.conf` drives the rebuild-on-kernel-update:

```ini
PACKAGE_NAME="nvidia-open"
PACKAGE_VERSION="610.xx"
AUTOINSTALL="yes"
MAKE[0]="make modules KERNEL_UODATE=$kernelver"
BUILT_MODULE_NAME[0]="nvidia"
BUILT_MODULE_NAME[1]="nvidia-modeset"
BUILT_MODULE_NAME[2]="nvidia-drm"
BUILT_MODULE_NAME[3]="nvidia-uvm"
DEST_MODULE_LOCATION[0]="/kernel/drivers/video"
```

```bash
sudo dkms add    -m nvidia-open -v 610.xx
sudo dkms build  -m nvidia-open -v 610.xx
sudo dkms install -m nvidia-open -v 610.xx
```

After install (or any module-option change), regenerate the initramfs so the
options are present at module load:

```bash
sudo mkinitcpio -P     # Arch; use dracut/update-initramfs on other distros
```

## Module options (`/etc/modprobe.d/nvidia.conf`)

The high-value knobs for a desktop/gaming box on the open driver:

```ini
# DRM modeset — REQUIRED for Wayland and smooth rendering
options nvidia_drm modeset=1

# Page Attribute Table — better memory throughput
options nvidia NVreg_UsePageAttributeTable=1

# Skip zeroing sysmem allocations — small perf win, low risk on desktop
options nvidia NVreg_InitializeSystemMemoryAllocations=0

# Low-latency GPU interrupt locking — helps desktop/gaming latency
options nvidia NVreg_RegistryDwords="RMIntrLockingMode=1"

# Resizable BAR (if the platform/firmware supports it)
options nvidia NVreg_EnableResizableBar=1

# GSP firmware — see the GSP note; on open this is usually required
options nvidia NVreg_EnableGpuFirmware=1

# Preserve VRAM across suspend/resume + multi-monitor framebuffer
options nvidia NVreg_PreserveVideoMemoryAllocations=1

# Allow unsupported GPUs / running under a VM with open modules
options nvidia NVreg_OpenRmEnableUnsupportedGpus=1
```

> [!warning]
> `NVreg_EnableGpuFirmware` is the GSP toggle. On the **open** driver GSP is
> generally *required* — setting it to `0` can break the module entirely. The
> classic "disable GSP for stability" advice is mostly a proprietary-driver
> story. See [[NVIDIA GSP Firmware]] before flipping it.

## nvidia-smi — the daily driver

```bash
nvidia-smi                       # snapshot: util, VRAM, temp, power, processes
nvidia-smi -l 1                  # refresh every second
nvidia-smi dmon                  # scrolling device monitor (sm/mem/enc/dec)
nvidia-smi -q -d POWER,CLOCK     # detailed power + clock query
nvidia-smi --query-gpu=name,memory.used,memory.total,utilization.gpu \
           --format=csv          # scriptable CSV
nvidia-smi -pl 400               # set power limit (watts) — tuning headroom
nvidia-smi -pm 1                 # persistence mode on
```

`nvidia-smi` reports the driver + CUDA version it was built against — first stop
when a CUDA workload can't see the GPU.

## NVIDIA on Wayland

Running [[Wayland]] on NVIDIA was historically the rough edge of the Linux
desktop. Modern RTX (40/50-series) on a recent kernel + nvidia-open is largely
fine; older cards (20/30-series) need more care and are sometimes better served
by [[VFIO GPU Passthrough]] + Looking Glass.

### Explicit sync — the big fix

The screen-tearing/stutter era ended with **explicit sync**, now implemented by
KWin, GNOME, and recent Hyprland. Requirements:

- Recent driver (590+/610 open) and kernel 6.1+ (7.x ideal)
- DRM modeset on (above)
- `__GLX_VENDOR_LIBRARY_NAME=nvidia` in the session environment

```bash
# session env (e.g. /etc/environment or compositor env)
__GLX_VENDOR_LIBRARY_NAME=nvidia
LIBVA_DRIVER_NAME=nvidia          # VA-API → NVDEC
# legacy escape hatch on some wlroots compositors:
# WLR_NO_HARDWARE_CURSORS=1
```

VRR / G-Sync and frame-pacing launch env on wlroots-style compositors:

```bash
__GL_GSYNC_ALLOWED=1 __GL_VRR_ALLOWED=1 __GL_YIELD="USLEEP"
```

### VRR (variable refresh rate)

VRR / G-Sync works well on Wayland with the open driver. Enable it in the
compositor (KDE: **System Settings → Display → Adaptive sync → Automatic/Always**)
and opt games in at launch:

```bash
__GL_GSYNC_ALLOWED=1 __GL_VRR_ALLOWED=1 %command%
```

Pair VRR with Gamescope (`gamescope --adaptive-sync` / `-r <max-hz>`) so the game
drives the refresh window directly. Align multi-monitor refresh rates — a
mismatched second display is a classic cause of stutter and the frozen-monitor bug.

### HDR

KDE Plasma 6 + KWin has working HDR on the open driver; enable it per-display in
**System Settings → Display & Monitor → HDR**, then opt apps/games in (Gamescope
`--hdr-enabled`, Proton `PROTON_ENABLE_WAYLAND=1 PROTON_ENABLE_HDR=1`). Compositor
HDR is where this lands — see [[KDE Plasma]] for the desktop-side toggles and
color management.

### Topics that still bite

- HDR tone-mapping varies by app/compositor; SDR-in-HDR brightness can need tuning
- Hybrid iGPU/dGPU handoff on laptops
- PipeWire screenshots/screencasts on some stacks
- Per-compositor quirks — a KWin fix may not apply to Hyprland/Sway

### Frozen-monitor workaround (KDE + Wayland)

A known bug freezes the monitor while the session is actually still alive. Don't
reboot — bounce the VT:

```
Ctrl + Alt + F3     # switch to a TTY
Ctrl + Alt + F1     # switch back to the graphical session
```

The session isn't killed, just refreshed. The issue is reduced on kernels that
run GSP at lower frequency — see [[NVIDIA GSP Firmware]].

## Digital vibrance & color

NVIDIA's "digital vibrance" (saturation boost, beloved by FPS players) has no
native Wayland control panel. Options:

- **[[nvcontrol]]** (`nvctl`) — a Rust GPU control tool that exposes digital
  vibrance, fan/clock, and more on Wayland where `nvidia-settings` falls short.
- **Gamescope** has a built-in saturation knob — a tiny Lua config bumps it
  per-session:

```lua
-- gamescope saturation (~1.25 = punchy without clipping)
return { display = { colorSaturation = 1.25 } }
```

## Gaming

### Gamescope wrapper

Gamescope is the micro-compositor for games — it isolates the game's present
loop from the desktop compositor, enabling proper VRR, scaling, and HDR. A
launcher that forces the NVIDIA Vulkan ICD and GBM backend:

```bash
#!/bin/bash
WLR_NO_HARDWARE_CURSORS=1 \
VK_ICD_FILENAMES=/usr/share/vulkan/icd.d/nvidia_icd.json \
VK_LAYER_PATH=/usr/share/vulkan/explicit_layer.d \
__GLX_VENDOR_LIBRARY_NAME=nvidia \
GBM_BACKEND=nvidia-drm \
gamescope -e -- "$@"
```

Use in Steam as a launch option: `gamescope-nvidia %command%` (or call
`gamescope -W 3840 -H 2160 -r 240 -- %command%` directly for explicit
resolution/refresh).

### Vulkan heap descriptors + Proton Experimental (DX12)

The current sweet spot for DX12 titles on NVIDIA: **Proton Experimental** (or a
recent Proton-GE) paired with NVIDIA's newer **Vulkan descriptor-buffer / heap
descriptor** path. VKD3D-Proton translates DX12 → Vulkan, and the heap-descriptor
path cuts binding/descriptor overhead — the gap to Windows on DX12 titles is now
small, sometimes reversed. Set it **per-game** in Steam launch options:

```bash
# Steam launch options — Proton Experimental, optimized DX12 path
PROTON_USE_NTSYNC=1 \
VKD3D_CONFIG=dxr11 \
DXVK_ENABLE_NVAPI=1 \
%command%
```

- Select **Proton Experimental** in the title's Compatibility settings (it ships
  the freshest DXVK/VKD3D-Proton + NTSYNC support).
- `VKD3D_CONFIG=dxr` / `dxr11` enables DXR ray tracing for DX12 titles.
- The heap-descriptor improvements ride in the driver + VKD3D-Proton version, so
  keep both current rather than chasing one magic flag.

> [!note]
> Vulkan/Proton flags move fast across driver and Proton releases. Prefer
> **per-game** launch options over global env so a flag that helps one title
> can't regress another. When in doubt, Proton Experimental + a current driver
> beats hand-tuned flags on an old stack.

### DLSS

DLSS (upscaling + frame generation) works under Proton via DXVK-NVAPI, which
re-exposes NVAPI so Windows games can negotiate DLSS on Linux:

```bash
# enable NVAPI/DLSS reporting to the game
DXVK_ENABLE_NVAPI=1 PROTON_ENABLE_NVAPI=1 %command%
```

Proton-GE bundles current DXVK-NVAPI; recent stock Proton does too. Frame
generation needs a 40/50-series card and a Proton build new enough to advertise it.

### "Unsafe" GPU launch params (browsers)

Hardware-accelerated WebGPU/WebGL and video decode in Chromium browsers
(e.g. **Brave**) often need flags to use the NVIDIA GPU on Wayland. The
`#enable-unsafe-webgpu` flag and Vulkan/ANGLE backends unlock acceleration that
is gated by default:

```bash
brave --enable-features=Vulkan,UseOzonePlatform \
      --ozone-platform=wayland \
      --enable-unsafe-webgpu \
      --use-angle=vulkan
```

"Unsafe" is NVIDIA/Chromium's caution label for not-yet-stable GPU paths, not a
security setting — it simply opts into acceleration the browser won't enable on
its own.

## AI / CUDA tuning

The NVIDIA stack is one CUDA toolchain delivered three ways — native on the
desktop, inside a VFIO guest, and **never baked into a container image** (the
[[NVIDIA Container Toolkit]] injects it at runtime). Tuning notes for local
inference:

- **Power/clocks** — cap power with `nvidia-smi -pl <watts>` and lock clocks
  (`nvidia-smi -lgc <min>,<max>`) for steady tokens/sec and lower heat on long
  runs.
- **Persistence mode** (`nvidia-smi -pm 1`) avoids driver re-init latency
  between jobs.
- **VRAM is the constraint** — quantize (GGUF Q4/Q5, AWQ/GPTQ) to fit larger
  models; `nvidia-smi --query-gpu=memory.used` to watch headroom.
- **Native vs container** — latency-critical single-tenant inference (e.g.
  `ollama-cuda`) runs best native; many concurrent GPU workloads or CI fan out
  via the container toolkit. Whole-GPU isolation → [[VFIO GPU Passthrough]].
- **Frameworks** — PyTorch (cu12x), vLLM, llama.cpp/Ollama, ComfyUI all consume
  the same driver; match the CUDA build to the installed driver (`nvidia-smi`).

## Verifying the stack

```bash
nvidia-smi                                   # driver loaded, GPU visible
cat /sys/module/nvidia_drm/parameters/modeset  # should print Y
echo $XDG_SESSION_TYPE                        # wayland
vulkaninfo | grep -i "deviceName\|driverName" # NVIDIA Vulkan ICD active
__NV_PRIME_RENDER_OFFLOAD=1 glxinfo | grep "OpenGL renderer"
nvidia-smi -q -d POWER                        # power-limit headroom for tuning
```

## Strengths / trade-offs

**Strengths**
- nvidia-open + recent kernel makes Wayland + RTX genuinely smooth.
- One CUDA toolchain across bare-metal, VMs, and containers.
- Per-game launch options give fine control over Vulkan/DLSS without global risk.

**Trade-offs**
- Userspace is still proprietary; behaviour shifts driver-to-driver, so notes
  rot quickly (re-verify per driver/Proton release).
- GSP firmware remains the root of a class of hangs (see
  [[NVIDIA GSP Firmware]]).
- 20/30-series on Wayland can be more work than they're worth — VFIO is often
  the cleaner path.

## Related

- [[NVIDIA GSP Firmware]] — GSP behaviour, throttling/disabling, the hang class
- [[NVIDIA Container Toolkit]] — GPU in containers (its own concern)
- [[Modprobe Options]] — how module parameters are set and persisted
- [[Wayland]] — the display protocol · [[KDE Plasma]] — the desktop most affected
- [[VFIO GPU Passthrough]] — whole-GPU isolation for guests / older cards
- [[nvcontrol]] — Wayland GPU control (digital vibrance, fan/clock)
