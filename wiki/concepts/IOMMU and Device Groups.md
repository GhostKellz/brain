---
type: concept
title: "IOMMU and Device Groups"
created: 2026-06-21
updated: 2026-06-21
tags:
  - virtualization
  - iommu
  - vfio
  - pcie
status: seed
related:
  - "[[VFIO GPU Passthrough]]"
  - "[[CUDA Delivery Paths]]"
---

# IOMMU and Device Groups

The **IOMMU** (Intel VT-d / AMD-Vi) is the hardware unit that makes safe GPU
passthrough possible. It sits between PCIe devices and system memory, translating
device DMA addresses — and, crucially for [[VFIO GPU Passthrough|VFIO]], it
**groups devices** into the smallest sets that can be isolated from each other.

> [!key-insight]
> You can only pass through a **whole IOMMU group** at once. The layout of those
> groups — decided by the motherboard and CPU — is what makes passthrough clean
> or painful. Buying silicon with good IOMMU separation up front (each PCIe slot
> and major controller in its own group) avoids the ACS-override hacks that
> weaken isolation later.

## What an IOMMU group is

A group is the **smallest set of devices the IOMMU can isolate as a unit**. If the
GPU shares a group with, say, a USB controller, then passing the GPU to a VM means
the USB controller goes with it — the host can't keep it.

A **clean group** for passthrough holds only the GPU and its own HDMI/DisplayPort
audio function, and nothing else. That is the goal.

## Why grouping matters for VFIO

The decision tree when preparing a card for passthrough:

| Situation | Outcome |
|-----------|---------|
| GPU alone in its group | Bind the group to `vfio-pci`, pass the GPU only, host keeps everything else |
| GPU shares with USB/NVMe/etc. | The **whole group** must move to the guest — host loses those devices |
| Group is dirty, slot is movable | Relocate the GPU to a slot with its own group |
| Group is dirty, nothing else works | ACS override patch (splits groups artificially, weakens isolation) |

The right answer is almost always **better slot placement or hardware**, not ACS
override.

## Enabling IOMMU

Add to the kernel command line:

| Platform | Parameters |
|----------|------------|
| AMD      | `amd_iommu=on iommu=pt` |
| Intel    | `intel_iommu=on iommu=pt` |

`iommu=pt` (pass-through mode) leaves host-owned devices unmediated for performance
and only enforces address translation where it is actually needed.

## Inspecting your groups

```bash
#!/usr/bin/env bash
# List every IOMMU group and the devices in it
shopt -s nullglob
for g in /sys/kernel/iommu_groups/*/devices/*; do
  grp=$(basename "$(dirname "$(dirname "$g")")")
  printf 'IOMMU Group %s: ' "$grp"
  lspci -nns "${g##*/}"
done | sort -V
```

Read the output like this:

- The **GPU and its audio function** should share one group and ideally nothing
  else — that group is what you bind to `vfio-pci`.
- If unrelated controllers (USB, NVMe, SATA) appear in the same group, that is the
  warning sign that the slot or board groups poorly.

## ACS override (last resort)

When a board lumps too much into one group, the ACS-override kernel patch can split
groups artificially:

```
pcie_acs_override=downstream,multifunction
```

Trade-offs:

- **Pro:** lets you isolate a GPU on a board that otherwise cannot.
- **Con:** the split is **not enforced by hardware** — devices the IOMMU *thinks*
  are isolated may still DMA to each other. Acceptable for a homelab with trusted
  guests, **not** as a real security boundary.

Prefer, in order: (1) a different PCIe slot, (2) a different board/CPU with native
separation, (3) ACS override only if neither is possible.

> [!gap]
> The specific IOMMU group map for any given machine (which devices land in which
> group) is hardware-specific and lives in the private tier.

## Related

- [[VFIO GPU Passthrough]] — bind the group and build the guest
- [[CUDA Delivery Paths]] — where VFIO fits among the ways to deliver a GPU
