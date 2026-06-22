---
type: reference
title: "Windows Administration"
created: 2026-06-21
updated: 2026-06-21
tags:
  - windows
  - powershell
  - winget
  - dism
status: developing
related:
  - "[[Windows Activation]]"
  - "[[Active Directory Administration]]"
  - "[[Microsoft 365 Administration]]"
  - "[[Endpoint Security Tooling]]"
---

> [!key-insight] Generalized from field notes; host/client-specific values are placeholders.

Windows desktop/server administration: package management, updates, repair, power, boot, and local accounts.

## Package Management (winget)

```powershell
winget update --all
winget update --all --include-unknown
winget upgrade --all --accept-package-agreements
winget search <app>
winget install -e --id <publisher.app>     # e.g. tailscale.tailscale
```

If winget is unavailable in PowerShell, call the full path under `WindowsApps`, or re-register the package:

```powershell
Add-AppxPackage -RegisterByFamilyName -MainPackage Microsoft.DesktopAppInstaller_8wekyb3d8bbwe
```

## Windows Updates (PSWindowsUpdate)

```powershell
Set-ExecutionPolicy -ExecutionPolicy Unrestricted
Install-Module -Name PSWindowsUpdate
Import-Module -Name PSWindowsUpdate

# Install with / without reboot
Get-WindowsUpdate -install -acceptall -autoreboot
Get-WindowsUpdate -install -acceptall -IgnoreReboot

# Hide / unhide / uninstall specific KBs
$HideList = "KB5005565","KB5005566"
Get-WindowsUpdate -KBArticleID $HideList -Hide
Get-WindowsUpdate -KBArticleID $HideList -WithHidden -Hide:$false
Remove-WindowsUpdate -KBArticleID KB5005565 -NoRestart
wusa /uninstall /kb:5005565

# Install an individual patch
Get-WindowsUpdate -KBArticleID KB890830 -install
```

When run through an unattended RMM/remote terminal, set a long timeout and reset execution policy afterward:

```powershell
Set-ExecutionPolicy Restricted    # restore after the run
```

### Scheduled reboots

```cmd
shutdown /r /t 10800      :: 3 hours
shutdown /r /t 21600      :: 6 hours
shutdown -r -f -t 00      :: immediate, forced
```

## System Repair (DISM / SFC)

```cmd
dism /online /cleanup-image /checkhealth
dism /online /cleanup-image /scanhealth
dism /online /cleanup-image /restorehealth
sfc /scannow

:: Use offline WIM as the source when WU is unavailable
DISM /Online /Cleanup-Image /RestoreHealth /Source:E:\Sources\install.wim
Get-WindowsImage -ImagePath "D:\sources\install.wim"

chkdsk /f /r C:
```

### Boot repair / BCD

```cmd
bootrec /rebuildbcd
bootrec /fixmbr
bootrec /fixboot
bcdedit /set {default} device partition=c:
bcdedit /set {default} osdevice partition=c:
```

## Power Management

```cmd
powercfg /getactivescheme
powercfg /setactive 8c5e7fda-e8bf-4a96-9a85-a6e23a8c635c   :: High Performance
powercfg /hibernate off
powercfg -h off                                            :: disable Fast Startup
```

## Networking & Profiles

```powershell
Get-NetConnectionProfile
Set-NetConnectionProfile -Name "<network>" -NetworkCategory Private   # Public/Private/DomainAuthenticated
Set-NetConnectionProfile -Name "<old>" -NewName "<new>"

netsh interface set interface "<adapter>" disable
netsh wlan show interfaces
netsh wlan show wlanreport
netsh advfirewall set allprofiles state on   # / off
```

## Local Accounts (CLI)

```cmd
net localgroup Administrators                     :: list local admins
net user /add <user> <password>
net localgroup administrators <user> /add
net localgroup administrators <user> /delete
```

Set a local account's password to never expire:

```cmd
WMIC USERACCOUNT WHERE Name='<user>' SET PasswordExpires=FALSE
```

## Misc Diagnostics

```powershell
query user                                  # logged-in sessions
(get-date) - (gcim Win32_OperatingSystem).LastBootUpTime    # uptime
systeminfo | find "System Boot Time"
wmic qfe list                               # installed updates
gci -r | sort -descending -property length | select -first 10 name,length,directory   # 10 largest files

# Force TLS 1.2 for older Windows when scripts make web requests
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12

# File hash
CertUtil -hashfile "<path>" SHA256
```

## Feature Enablement

```powershell
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All

# WSL + Hyper-V + VM Platform in one line
dism.exe /online /enable-feature /featurename:Microsoft-Hyper-V-All /featurename:VirtualMachinePlatform /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart

# RSAT tools
Get-WindowsCapability -Name RSAT* -Online | Add-WindowsCapability -Online
```

## Windows 11 Requirement Bypass

Registry keys to bypass TPM/Secure Boot/CPU checks (use only where you have a legitimate reason):

```
HKLM\SYSTEM\Setup\LabConfig
  BypassTPMCheck        (DWORD) = 1
  BypassSecureBootCheck (DWORD) = 1
HKLM\SYSTEM\Setup\MoSetup
  AllowUpgradesWithUnsupportedTPMOrCPU (DWORD) = 1
```

## Notes

- Activation lives in [[Windows Activation]].
- Domain join, GPO, and AD account ops: [[Active Directory Administration]].
- Defender and EDR tooling: [[Endpoint Security Tooling]].
