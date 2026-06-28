---
type: reference
title: "Huntress"
created: 2026-06-28
updated: 2026-06-28
tags:
  - security
  - edr
  - itdr
  - siem
  - mdr
  - defender
  - msp
status: developing
related:
  - "[[Endpoint Security Tooling]]"
  - "[[Microsoft 365 Services]]"
  - "[[Windows Administration]]"
  - "[[Exchange Online Administration]]"
  - "[[CK Technology LLC]]"
---

> [!key-insight] Huntress is **managed** security, not just software. Every
> product ships with a 24/7 human + AI-assisted SOC that triages and remediates —
> the value is that alerts are rare, real, and actionable instead of a firehose
> you have to staff. That "quiet" model is the whole point: you only hear from it
> when something matters.

**Huntress** is an SMB/MSP-focused managed security platform. It catches the
foothold *after* the initial compromise — persistence, lateral movement, identity
abuse — and a SOC remediates it, rather than expecting the customer to read raw
telemetry. Account keys, org keys, and tenant identifiers are **secrets** — never
put real values in a public note; pull them from the RMM/secrets store at deploy
time.

## The product line

| Product | Protects | What it does |
|---------|----------|--------------|
| **Managed EDR** | Windows / macOS / Linux endpoints | Behavioral detection of persistent footholds + SOC remediation (Huntress-built agent, not a reseller) |
| **Managed Antivirus / Microsoft Defender** | Windows endpoints | Centrally manages Defender AV config, exclusions, scans — **at no extra cost** alongside EDR |
| **Managed ITDR** | Microsoft 365 / Google Workspace identities | Account takeover, BEC, session hijacking, rogue OAuth apps — "identity is the new endpoint" |
| **Managed SIEM** | Endpoint / identity / firewall logs | Smart-filtered log capture + retention for compliance, same SOC layer |
| **SAT** | Users | Security Awareness Training |
| **ESPM** | Endpoint posture | Hardening: configs, apps, vulnerabilities visibility |

> [!key-insight] EDR + Managed Defender is the signature combo. Huntress's EDR
> doesn't replace Microsoft Defender Antivirus — it *manages and improves* it
> (recommended configs, risky-exclusion monitoring) while adding the behavioral
> EDR + SOC layer on top. Cheaper than a third-party AV and better-integrated. The
> catch: Defender has to actually be **on** and configured correctly, which is why
> the PowerShell enablement below matters.

## Agent deployment

Most managed-security agents (Huntress included) install via a vendor PowerShell
script that takes an **account key** and **org key**, optionally with **tags** for
RMM grouping. Treat both keys as secrets.

```powershell
# Huntress-style installer; supply keys/tags from a secret store, never hard-coded
powershell -executionpolicy bypass -f .\InstallHuntress.powershellv2.ps1 `
  -acctkey <account-key> -orgkey <org-key> -tags <tags> -reregister -reinstall
```

Confirm the agent's listening process / outbound connections:

```powershell
Get-NetTcpConnection |
  Select Local*,Remote*,State,@{Name="Process";Expression={(Get-Process -Id $_.OwningProcess).ProcessName}} |
  Where-Object { $_.Process -eq "HuntressAgent" }
```

> [!note] Huntress (like all EDR vendors) publishes a firewall allow-list —
> outbound `443` to its domains/CDN/S3 buckets. Pull the current list from the
> vendor docs rather than hard-coding it; it changes.

## Enabling Microsoft Defender via PowerShell

*(Huntress Support: EDR → Managed Antivirus → Status, Policy, and Configuration →
"Enable Microsoft Defender via PowerShell".)*

Huntress **Managed Antivirus / Managed Microsoft Defender** can only manage
Defender if Defender's real-time protection is actually running. **The Huntress
agent cannot enable Defender if it has been *explicitly* disabled.** Normally
Defender is on by default in Windows 8.1+ / Server 2016+ and even re-enables
itself when it detects no third-party AV — so if it's staying off, something is
explicitly disabling it (a leftover GPO/policy key or another AV), and that's what
the commands below clear.

> [!warning]
> These commands do **no error checking or install validation** — PowerShell
> errors can be terse. If you're not comfortable troubleshooting them, enable
> Defender through the **GUI** instead. Also:
> - They **won't work if Windows Defender isn't installed** on the machine.
> - Running them while **another AV is installed** can cause conflicts between the two.
> - Take caution: if Defender refuses to enable, you may have another underlying issue present.

Best practice is to set this up as a **script in your RMM** for easy, repeatable
enablement. If running manually, run the lines **one at a time, in order** — they
depend on the registry key being created before the properties are set.

```powershell
# Re-enable real-time + on-access protection
Set-MpPreference -DisableRealtimeMonitoring $false
Set-MpPreference -DisableIOAVProtection $false

# Clear the policy keys a prior AV / GPO may have set to disable Defender
New-Item -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows Defender" -Name "Real-Time Protection" -Force
New-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows Defender\Real-Time Protection" -Name "DisableBehaviorMonitoring" -Value 0 -PropertyType DWORD -Force
New-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows Defender\Real-Time Protection" -Name "DisableOnAccessProtection" -Value 0 -PropertyType DWORD -Force
New-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows Defender\Real-Time Protection" -Name "DisableScanOnRealtimeEnable" -Value 0 -PropertyType DWORD -Force
New-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows Defender" -Name "DisableAntiSpyware" -Value 0 -PropertyType DWORD -Force

# Start the services
Start-Service WinDefend
Start-Service WdNisSvc
```

Verify state afterwards:

```powershell
Get-MpComputerStatus       # RealTimeProtectionEnabled should be True
Get-MpPreference           # confirm exclusions / settings
Start-MpScan -ScanType QuickScan
```

> [!key-insight] Why the registry keys, not just `Set-MpPreference`: a prior AV
> often disables Defender through **policy** keys under
> `HKLM\...\Policies\Microsoft\Windows Defender`. Those override the
> `Set-MpPreference` cmdlets, so Defender stays off until the policy DWORDs are
> zeroed — which is exactly what the block above does.

### Windows Server

Servers differ from client OSes — the `Set-MpPreference` block above may not be
enough. First confirm Defender isn't disabled via Group Policy or registry (per
Microsoft's third-party-migration troubleshooting). Then:

**If Defender is present but disabled** — re-enable via the `MpCmdRun` tool. On
Server 2016 you may specifically need `-WdEnable`. From an elevated prompt, run
the tool from the latest platform version, then reboot:

```cmd
:: cd into the newest Defender platform dir (falls back to Program Files)
cd /d "%ProgramData%\Microsoft\Windows Defender\Platform\<antimalware-platform-version>"
MpCmdRun.exe -WdEnable
:: then restart the device
```

**If the Defender *feature* was uninstalled/removed** — reinstall it with DISM:

```cmd
:: Windows Server 2016
Dism /Online /Enable-Feature /FeatureName:Windows-Defender-Features
Dism /Online /Enable-Feature /FeatureName:Windows-Defender
Dism /Online /Enable-Feature /FeatureName:Windows-Defender-Gui

:: Windows Server 1803 / 2019 / later
Dism /Online /Enable-Feature /FeatureName:Windows-Defender
```

On Server 2016, if the feature files were stripped, you may also need to
*Configure a Windows Repair Source* to restore the installation files first.

### Support limitations

If Defender still won't enable after the above, it's a Defender/OS problem, not a
Huntress one — escalate to **Microsoft Support**. Reference:

- Huntress — *Enable Microsoft Defender via PowerShell*:
  `support.huntress.io/hc/en-us/articles/4402989131283`
- Microsoft — *Enable and update Defender Antivirus to the latest version on Windows Server*:
  `learn.microsoft.com/en-us/defender-endpoint/enable-update-mdav-to-latest-ws`
- Microsoft — *Turn on Microsoft Defender Antivirus*.

## Where it fits

- General Defender PowerShell + EDR deployment patterns: [[Endpoint Security Tooling]].
- Identity side (ITDR) overlaps with [[Microsoft 365 Services]] / [[Exchange Online Administration]].
- Part of the standard managed-services stack at [[CK Technology LLC]].

## Related

- [[Endpoint Security Tooling]] — broader Defender/EDR patterns
- [[Microsoft 365 Services]] — what Managed ITDR protects
- [[Windows Administration]] — patch/host management alongside the agent
