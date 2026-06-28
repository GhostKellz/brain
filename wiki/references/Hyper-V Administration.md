---
type: reference
title: "Hyper-V Administration"
created: 2026-06-21
updated: 2026-06-21
tags:
  - hyper-v
  - virtualization
  - windows
status: developing
related:
  - "[[Windows Administration]]"
  - "[[Proxmox]]"
  - "[[Veeam]]"
---

> [!key-insight] Generalized from field notes; host/client-specific values are placeholders.

Hyper-V VM disk management: extending virtual disks and reclaiming space. Replace `<vm>` and disk paths.

## Enable Hyper-V

```powershell
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All
```

## Extend / Shrink Partition Inside a Guest (diskpart)

```
diskpart
list volume
list partition
select partition <n>
delete partition override     :: e.g. to remove a trailing recovery partition before extending
```

## Reclaim Space from a Dynamic VHDX

After freeing space inside the guest, compact the VHDX from the host:

```powershell
# 1. Shut the VM down first.

# 2. Detach the disk
Remove-VMHardDiskDrive -VMName "<vm>" -ControllerType SCSI -ControllerNumber 0 -ControllerLocation 1

# 3. Optimize (reclaim) the VHDX
Optimize-VHD -Path "<path>\<disk>.vhdx" -Mode Full

# 4. Re-attach
Add-VMHardDiskDrive `
  -VMName "<vm>" `
  -ControllerType SCSI -ControllerNumber 0 -ControllerLocation 1 `
  -Path "<path>\<disk>.vhdx"
```

## Notes

- Host OS tasks: [[Windows Administration]].
- For the Linux/KVM equivalent hypervisor, see [[Proxmox]].
- Image-level backup/DR of Hyper-V VMs (RCT incrementals, SOBR, offsite to
  cloud): [[Veeam]]. File-level + lighter VM/image coverage: [[Comet Backup]].
- The VHDX volume / Veeam repo filesystem (CoW, block cloning, 64K clusters):
  [[ReFS]].
