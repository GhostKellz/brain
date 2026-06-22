---
type: reference
title: "Microsoft 365 Administration"
created: 2026-06-21
updated: 2026-06-21
tags:
  - m365
  - azure-ad
  - powershell
  - licensing
status: developing
related:
  - "[[Exchange Online Administration]]"
  - "[[Email DNS Security]]"
  - "[[Active Directory Administration]]"
---

> [!key-insight] Generalized from field notes; host/client-specific values are placeholders.

Microsoft 365 / Entra ID tenant administration via PowerShell. All UPNs, tenant names, group/user ObjectIDs, and passwords below are placeholders — never commit real ones.

## Connect

```powershell
Install-Module ExchangeOnlineManagement
Connect-ExchangeOnline
Connect-MsolService          # legacy MSOnline (Entra) module
Connect-AzureAD              # legacy AzureAD module
```

> [!note] MSOnline and AzureAD modules are deprecated; prefer Microsoft.Graph PowerShell for new work. The commands here reflect legacy field usage.

## Licensing

```powershell
Get-MsolAccountSku                         # list SKUs available in the tenant

# Users assigned a specific license (match on AccountSkuId)
Get-MsolUser | Where-Object {($_.licenses).AccountSkuId -match "<SKU-PART>"}
```

## User Onboarding

```powershell
Connect-MsolService

New-MsolUser -UserPrincipalName "<user>@<tenant>.onmicrosoft.com" `
  -DisplayName "<Display Name>" -FirstName "<First>" -LastName "<Last>" `
  -UsageLocation "US" -LicenseAssignment "<tenant>:<SKU>"

# Set password (force change at next sign-in as appropriate)
Set-MsolUserPassword -UserPrincipalName "<user>@<tenant>.onmicrosoft.com" `
  -NewPassword "<temp-password>" -ForceChangePassword $true

# Remove a mistakenly-created account
Remove-MsolUser -UserPrincipalName "<user>@<tenant>.onmicrosoft.com" -Force
```

## Groups

```powershell
Get-MsolGroup                               # find group ObjectIDs
Get-MsolUser -UserPrincipalName "<user>@<tenant>.onmicrosoft.com" | Select-Object * | Format-List

# Add a user to a security group (GroupObjectId + member GroupMemberObjectId)
Add-MsolGroupMember -GroupObjectId <group-object-id> `
  -GroupMemberType User -GroupMemberObjectId <member-object-id>

# Add a user to a Microsoft 365 (unified) group
Add-UnifiedGroupLinks -Identity "<Group Name>" -LinkType Members `
  -Links <user>@<tenant>.onmicrosoft.com -Confirm
```

## Conditional Access — Named Location (example)

```powershell
Connect-AzureAD
$OfficeIP = Read-Host "Enter the office public IP (e.g. <ip>/32)"
New-AzureADMSNamedLocation -DisplayName "Main Office" -Locations $OfficeIP
```

## Viva / Briefing Emails

```powershell
Get-UserBriefingConfig
Set-UserBriefingConfig -Identity <user>@<tenant> -Enabled $false

# Disable for all users
$users = Get-User
$users | Foreach { Set-UserBriefingConfig -Identity $_.UserPrincipalName -Enabled $false }
```

## Notes

- Mailbox / mail-flow tasks: [[Exchange Online Administration]].
- DKIM/SPF/DMARC: [[Email DNS Security]].
