---
type: reference
title: "Exchange Online Administration"
created: 2026-06-21
updated: 2026-06-21
tags:
  - exchange-online
  - m365
  - email
  - powershell
status: developing
related:
  - "[[Microsoft 365 Administration]]"
  - "[[Email DNS Security]]"
  - "[[Endpoint Security Tooling]]"
---

> [!key-insight] Generalized from field notes; host/client-specific values are placeholders.

Exchange Online mailbox, mail-flow, and Defender-for-Office tasks via PowerShell. All addresses/domains are placeholders.

## Connect

```powershell
Set-ExecutionPolicy RemoteSigned
Install-Module ExchangeOnlineManagement -AllowClobber -Force
Connect-ExchangeOnline -UserPrincipalName <admin>@<tenant>.onmicrosoft.com
```

## Mailbox Permissions

```powershell
Get-MailboxPermission -Identity "<mailbox>"

# Grant full access of one mailbox to another user
Add-MailboxPermission -Identity "<mailbox>" -User "<delegate>" `
  -AccessRights FullAccess -InheritanceType All
```

## OWA Access Toggle

```powershell
Set-CASMailbox <user>@<domain> -OWAEnabled $false
Set-CASMailbox <user>@<domain> -OWAEnabled $true
```

## In-Place Archive

```powershell
Enable-Mailbox -Identity <user> -Archive          # per user
Set-OrganizationConfig -AutoExpandingArchive      # tenant-wide

# List users with an active archive
Get-Mailbox -ResultSize Unlimited -RecipientTypeDetails UserMailbox |
  Where-Object {$_.ArchiveStatus -eq "Active"} |
  Select DisplayName,ArchiveStatus
```

## Send-As Alias

```powershell
Get-OrganizationConfig | select SendFromAliasEnabled
Set-OrganizationConfig -SendFromAliasEnabled $true
```

## Distribution / Unified Groups

```powershell
# Export DL membership
Get-DistributionGroupMember -Identity "<dl>@<domain>" |
  Select-Object DisplayName,PrimarySmtpAddress |
  Export-Csv -Path "C:\members.csv" -NoTypeInformation

# Suppress the welcome email on a M365 group
Set-UnifiedGroup -Identity "<group>@<domain>" -UnifiedGroupWelcomeMessageEnable:$false
```

## Junk / Safe Senders

```powershell
Get-Mailbox -ResultSize Unlimited |
  Set-MailboxJunkEmailConfiguration -TrustedSendersAndDomains @{Add="<domain>","<sender>@<domain>"}
```

## Message Trace & Quarantine

```powershell
# Quarantine
Get-QuarantineMessage -SenderAddress "<sender>@<domain>"
Get-QuarantineMessage -SenderAddress "<sender>@<domain>" | Release-QuarantineMessage -ReleaseToAll
Get-QuarantineMessage -SenderAddress "<sender>@<domain>" | Out-File C:\quarantined.txt

# Message trace (date range)
Get-MessageTrace -SenderAddress <sender>@<domain> -RecipientAddress <rcpt>@<domain> `
  -StartDate <MM/DD/YYYY> -EndDate <MM/DD/YYYY>

# Last 10 days to file
$startDate = (Get-Date).AddDays(-10); $endDate = Get-Date
Get-MessageTrace -StartDate $startDate -EndDate $endDate -SenderAddress <sender>@<domain> |
  Select-Object Timestamp,SenderAddress,RecipientAddress,Status,Received,Subject |
  Out-File C:\trace.txt
```

## Meeting Rooms

```powershell
Set-MailboxCalendarConfiguration -Identity <room>@<domain> -WorkingHoursStartTime 07:00:00
Set-MailboxCalendarConfiguration -Identity <room>@<domain> -WorkingHoursTimeZone "Eastern Standard Time"
Get-MailboxCalendarConfiguration -Identity <room>@<domain>
```

## Defender for Office 365

```powershell
# Safe Attachments for SharePoint / OneDrive / Teams
Set-AtpPolicyForO365 -EnableATPForSPOTeamsODB $true
Get-ATPPolicyForO365

# View preset security policy rules
Get-ATPBuiltInProtectionRule
Get-EOPProtectionPolicyRule -Identity "Standard Preset Security Policy"
```

## Notes

- Tenant/user/licensing tasks: [[Microsoft 365 Administration]].
- DNS-level mail authentication: [[Email DNS Security]].
