---
type: concept
title: "CUDA Delivery Paths"
created: 2026-06-21
updated: 2026-06-21
tags:
  - nvidia
  - cuda
  - containers
  - virtualization
status: seed
related:
  - "[[NVIDIA Container Toolkit]]"
  - "[[VFIO GPU Passthrough]]"
  - "[[Local LLM Inference]]"
---

# CUDA Delivery Paths

There are three ways to get a CUDA workload onto an NVIDIA GPU. All three
converge on the same frameworks (PyTorch, vLLM, Ollama, `nvcc`), so code moves
between them unchanged — the difference is **where the driver lives** and the
isolation/sharing trade-off.

| Path | Driver location | Image | Best for |
|------|-----------------|-------|----------|
| **Native bare-metal** | host OS | n/a | daily driver, lowest latency, single tenant |
| **VFIO passthrough VM** | guest VM | n/a | hard isolation, a whole GPU per VM |
| **Container toolkit** | host/VM (injected) | driver-free | many workloads sharing one card, portability, CI |

> [!key-insight]
> The driver lives in exactly one place per context: natively on the desktop,
> inside the VFIO guest, and **never inside a container image** — the
> [[NVIDIA Container Toolkit]] injects it at run time.

## Native bare-metal

The driver and CUDA toolkit are installed on the host; processes use the GPU
directly. Lowest latency, simplest, but single-tenant — one OS owns the card.
This is why latency-critical single-tenant inference (an Ollama daily driver)
runs native rather than containerized. See [[Local LLM Inference]].

## VFIO passthrough VM

The host hides the GPU from itself (`vfio-pci`) and hands the whole card to a
guest VM, which installs its own driver. Full isolation; the guest behaves as if
it has a physical card. Add Looking Glass to use such a VM as a desktop. See
[[VFIO GPU Passthrough]].

## Container toolkit

The driver lives on the host (or inside a passthrough VM), and the
[[NVIDIA Container Toolkit]] injects GPU devices + driver libs into each
container at run time. Many containers fan out over one card; images stay
portable across whatever driver each node has.

A common cluster pattern stacks two of these: pass a GPU to a VM via VFIO, then
run the container toolkit *inside* that VM so many containers share the single
passed-through card.

## Choosing

- One tenant, latency matters → **native**.
- A guest needs the whole GPU / hard isolation → **VFIO**.
- Many workloads share a card, want portable images → **container toolkit**.

## Related

- [[NVIDIA Container Toolkit]] — the injection mechanism
- [[VFIO GPU Passthrough]] — the isolation path
- [[Local LLM Inference]] — why inference often stays native
