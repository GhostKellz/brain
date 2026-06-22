---
type: reference
title: "Active Directory Administration"
created: 2026-06-21
updated: 2026-06-21
tags:
  - active-directory
  - powershell
  - gpo
status: developing
related:
  - "[[Windows Administration]]"
  - "[[Microsoft 365 Administration]]"
---

> [!key-insight] Generalized from field notes; host/client-specific values are placeholders.

On-prem Active Directory account management, Group Policy, and related Windows server tasks. Replace `<user>` / `<domain>` with real values.

## Account Lockout / Unlock

```powershell
# Check lock status
Get-ADUser -Identity <user> -Properties LockedOut |
  Select-Object samAccountName,LockedOut | ft -AutoSize

# Show lockout timestamp
Get-ADUser <user> -Properties Name,lockoutTime |
  Select-Object Name,@{n='lockoutTime';e={[DateTime]::FromFileTime($_.lockoutTime)}}

# Unlock
Unlock-ADAccount <user> -Confirm
```

## Enable / Disable Accounts

```powershell
Disable-ADAccount -Identity <user>
Get-ADUser <user> | Select Name,Enabled       # verify
```

## Group Policy

```cmd
gpupdate /force            :: refresh policy now
gpresult /R                :: show applied policies (run as the user)
net use                    :: show mapped drives
tzutil /s "Eastern Standard Time"   :: set time zone via GPO/login script
```

> [!note] If group membership seems cached/delayed, investigating/purging Kerberos tickets (`klist purge`) is a reasonable diagnostic starting point.

## Scheduled Task via GPO

Pattern for deploying a maintenance script (e.g. updates) fleet-wide:

1. Store the PowerShell/batch script on a share, e.g. a SYSVOL or file-server UNC path: `\\<file-server>\<share>\<script>.ps1`.
2. Create a scheduled task (GPO preference) running as `SYSTEM` with a daily/weekly/monthly trigger.
3. Action -> Start a program:
   - Program: `C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe`
   - Arguments: `-ep bypass -f \\<file-server>\<share>\<script>.ps1`

## Remote Desktop Services (RDS) App Install

```cmd
change user /install     :: enter install mode before installing apps
change user /execute     :: return to execute mode afterward
```

## Notes

- Cloud identity (Entra/M365) account ops are in [[Microsoft 365 Administration]].
- Local (non-domain) account commands: [[Windows Administration]].
