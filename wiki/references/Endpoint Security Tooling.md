---
type: reference
title: "Endpoint Security Tooling"
created: 2026-06-21
updated: 2026-06-21
tags:
  - security
  - defender
  - edr
  - windows
status: developing
related:
  - "[[Windows Administration]]"
  - "[[Exchange Online Administration]]"
  - "[[CK Technology Solutions]]"
---

> [!key-insight] Generalized from field notes; host/client-specific values are placeholders.

Microsoft Defender management and third-party EDR/agent deployment patterns. Account keys, org keys, and onboarding scripts are secrets — never put real ones in a public note.

## Microsoft Defender (PowerShell)

```powershell
# Status / inspection
Get-MpComputerStatus
Get-MpPreference
Get-MpThreat
Get-MpThreatDetection

# Full scan
Start-MpScan -ScanType FullScan

# Schedule scans / remediation
Set-MpPreference -ScanParameters 2
Set-MpPreference -RemediationScheduleDay 4
Set-MpPreference -RemediationScheduleTime 21:00:00
```

### Re-enable Defender real-time protection

```powershell
Set-MpPreference -DisableRealtimeMonitoring $false
Set-MpPreference -DisableIOAVProtection $false
New-Item -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows Defender" -Name "Real-Time Protection" -Force
New-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows Defender\Real-Time Protection" -Name "DisableBehaviorMonitoring" -Value 0 -PropertyType DWORD -Force
New-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows Defender\Real-Time Protection" -Name "DisableOnAccessProtection" -Value 0 -PropertyType DWORD -Force
New-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows Defender\Real-Time Protection" -Name "DisableScanOnRealtimeEnable" -Value 0 -PropertyType DWORD -Force
New-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows Defender" -Name "DisableAntiSpyware" -Value 0 -PropertyType DWORD -Force
Start-Service WinDefend
Start-Service WdNisSvc
```

### Windows Defender Firewall

```powershell
Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled True   # or False to disable
netsh advfirewall set allprofiles state on                           # / off
```

## EDR Agent Deployment (Generic Pattern)

Most managed-security agents install via a vendor PowerShell/CMD script that takes an **account key** and **org key**. Treat both as secrets — pull them from your RMM/secrets store at deploy time, not from notes.

```powershell
# Generic shape (Huntress-style installer); supply keys/tags from a secret store
powershell -executionpolicy bypass -f .\InstallHuntress.powershellv2.ps1 `
  -acctkey <account-key> -orgkey <org-key> -tags <tags> -reregister -reinstall
```

Check listening ports owned by an agent process:

```powershell
Get-NetTcpConnection |
  Select Local*,Remote*,State,@{Name="Process";Expression={(Get-Process -Id $_.OwningProcess).ProcessName}} |
  Where-Object { $_.Process -eq "<AgentProcessName>" }
```

```bash
# macOS
sudo lsof -i -P | grep "<AgentName>"
```

> [!note] EDR vendors publish firewall allow-lists (typically outbound 443 to `*.<vendor-domain>` and their S3/CDN buckets). Pull the current list from the vendor's docs rather than hard-coding it.

## Other Defenses

```cmd
:: BitLocker
manage-bde -lock X:

:: Disable EFS file encryption
fsutil behavior set disableencryption 1

:: Block personal OneDrive sign-in (per-user policy)
reg add HKEY_CURRENT_USER\Software\Policies\Microsoft\OneDrive /v DisablePersonalSync /t REG_DWORD /d 1 /f
```

## Notes

- Update/patch workflows: [[Windows Administration]].
- Mail-side protection (Defender for Office 365): [[Exchange Online Administration]].
- These agent-deployment patterns are part of standard managed-services tooling at [[CK Technology Solutions]].
