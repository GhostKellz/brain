---
type: concept
title: "NVIDIA Container Toolkit"
created: 2026-06-21
updated: 2026-06-21
tags:
  - nvidia
  - docker
  - containers
  - cuda
status: seed
related:
  - "[[CUDA Delivery Paths]]"
  - "[[NVIDIA Container Runtime Troubleshooting]]"
  - "[[Local LLM Inference]]"
  - "[[VFIO GPU Passthrough]]"
---

# NVIDIA Container Toolkit

The **NVIDIA Container Toolkit** lets many containers share one GPU by
**injecting the GPU devices and driver libraries into the container at runtime** —
no driver is ever baked into the image. The same mechanism works on bare-metal
Linux and inside a passthrough VM.

```
docker run --gpus all ...
  └─ nvidia-container-runtime        (OCI runtime wrapper)
       └─ libnvidia-container        (mounts GPU devices + host driver libs in)
            └─ runc                   (runs the container)
```

> [!key-insight]
> The container image stays **driver-free and portable**. The toolkit injects
> whatever driver the *host* has at `docker run` time, so the same image runs on a
> 5090 desktop, a 4090 VM, or an older CI node with no rebuild.

## Why it's useful

- **One card, many tenants** — dozens of containers time-share a GPU instead of
  dedicating it to one process.
- **Driver/image decoupling** — upgrade the host driver without rebuilding images.
- **Reproducible CI** — test jobs run the exact image production runs.
- **Works inside passthrough VMs** — pairs with [[VFIO GPU Passthrough]] for
  isolation *and* fan-out.

## Quick start (Arch)

```bash
yay -S nvidia-container-toolkit
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker

# Smoke test — GPU visible inside a container
docker run --rm --gpus all nvidia/cuda:13.0-base nvidia-smi
```

## When to use it vs alternatives

```
Need a whole GPU isolated to one OS/VM?
  ├─ Yes → VFIO passthrough (+ Looking Glass for desktop)
  └─ No → Many workloads sharing one card?
            ├─ Yes → NVIDIA Container Toolkit
            └─ No (single, latency-critical) → native bare-metal
```

- **Native** — single-tenant, latency-critical inference (e.g. an Ollama daily
  driver). See [[Local LLM Inference]].
- **Container toolkit** — many concurrent GPU workloads, CI, portable images.
- **VFIO** — a guest needs the *whole* GPU and hard isolation. See
  [[VFIO GPU Passthrough]].

The three paths are compared in [[CUDA Delivery Paths]].

## The recurring breakage

After a driver update, GPU containers fail with `.so` path mismatches because the
CDI spec still references the old driver version. Fix workflow and a pacman hook
to automate it: [[NVIDIA Container Runtime Troubleshooting]].

## Related

- [[CUDA Delivery Paths]] — native vs VFIO vs container
- [[NVIDIA Container Runtime Troubleshooting]] — config + post-update fixes
- [[VFIO GPU Passthrough]] — the whole-GPU isolation path
