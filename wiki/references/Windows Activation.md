---
type: reference
title: "Windows Activation"
created: 2026-06-21
updated: 2026-06-21
tags:
  - windows
  - activation
  - kms
status: developing
related:
  - "[[Windows Administration]]"
  - "[[macOS Virtualization]]"
---

> [!key-insight] Generalized from field notes; host/client-specific values are placeholders.

KMS / GVLK activation workflow using `slmgr`, plus edition switching with DISM.

> [!note] Generic Volume License Keys (GVLKs) below are the **public** keys Microsoft publishes for KMS clients. They are not secret license keys and are documented in Microsoft's volume activation guide. They only activate against a KMS host you are licensed to use. Set `<kms-host>` to your own/your licensed KMS server.

## Public Generic GVLKs (Microsoft-published)

| Product | GVLK |
| --- | --- |
| Windows 10/11 Pro | `W269N-WFGWX-YVC9B-4J6C9-T83GX` |
| Windows Server 2022 Standard | `VDYBN-27WPP-V4HQT-9VMD4-VMK7H` |
| Windows Server 2025 Standard | `TVRH6-WHNXV-R9WG3-9XRFY-MY832` |
| Windows Server 2025 Datacenter | `D764K-2NDRG-47T6Q-P8T8W-YP6DF` |
| Windows Server 2025 Datacenter: Azure Edition | `XGN3F-F394H-FD2MY-PP6FD-8MCRC` |

The current, authoritative list lives in Microsoft's "KMS client setup keys" documentation.

## Activate via KMS (`slmgr`)

```powershell
# Install the product (GVLK) key
slmgr /ipk <GVLK>

# Point at your KMS host
slmgr /skms <kms-host>

# Activate
slmgr /ato
```

## Switch Server Edition (DISM)

Upgrade an installed Server SKU to a higher edition, then activate:

```powershell
DISM /online /Set-Edition:ServerStandard /ProductKey:<GVLK> /AcceptEula
slmgr /ipk <GVLK>
slmgr /skms <kms-host>
slmgr /ato
```

## Licensing Inspection

```powershell
slmgr /xpr           # activation status / expiry
slmgr.vbs /dli       # license info (pop-up)
slmgr.vbs /dlv       # detailed license info
slmgr -rearm         # reset activation grace timer (180 days)

# OEM key embedded in firmware
(Get-WmiObject -query 'select * from SoftwareLicensingService').OA3xOriginalProductKey
```

## Notes

- Edition switching and reboots tie into [[Windows Administration]].
- For activating macOS VMs you instead use the public OSK board key; see [[macOS Virtualization]].
