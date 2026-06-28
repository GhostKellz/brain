---
type: reference
title: "Email DNS Security"
created: 2026-06-21
updated: 2026-06-21
tags:
  - email
  - spf
  - dkim
  - dmarc
  - dns
status: developing
related:
  - "[[Microsoft 365 Administration]]"
  - "[[Exchange Online Administration]]"
  - "[[SMTP2GO]]"
---

> [!key-insight] Generalized from field notes; host/client-specific values are placeholders.

SPF, DKIM, and DMARC DNS records for Google Workspace and Microsoft 365 tenants. Replace `<domain>` and reporting mailboxes with your own.

## SPF (TXT record on `@`)

Google Workspace:

```
v=spf1 include:_spf.google.com ~all
```

Microsoft 365:

```
v=spf1 include:spf.protection.outlook.com -all
```

TTL: 3600 (1 hour) is fine.

## DKIM

### Google Workspace
Generate the DKIM key in the Admin console (Apps -> Google Workspace -> Gmail -> Authenticate email), then publish the supplied TXT record and turn on authentication.

### Microsoft 365
Publish two CNAME records pointing at the tenant's selector hosts (values come from the tenant; format shown):

```
selector1._domainkey   ->  selector1-<domain-dashed>._domainkey.<tenant>.onmicrosoft.com
selector2._domainkey   ->  selector2-<domain-dashed>._domainkey.<tenant>.onmicrosoft.com
```

Then enable DKIM signing in the Defender portal.

## DMARC (TXT record on `_dmarc`)

Roll out in phases — monitor, then quarantine a small percentage, then reject.

```
# Phase 1 — monitor only
v=DMARC1; p=none; rua=mailto:<reports>@<domain>

# Phase 2 — quarantine a small sample
v=DMARC1; p=quarantine; pct=5; rua=mailto:<reports>@<domain>

# Phase 3 — enforce
v=DMARC1; p=reject; rua=mailto:<reports>@<domain>; ruf=mailto:<reports>@<domain>; pct=100; adkim=s; aspf=s
```

> [!note] Some DNS providers auto-append the domain to the record name; verify whether you enter `_dmarc` or `_dmarc.<domain>`.

## Verification

- Microsoft 365: Admin Center -> Settings -> Domains -> DNS health check; DKIM status in the Defender portal.
- Google Workspace: built-in DKIM authentication status page.

## Notes

- Mail-flow troubleshooting (message trace/quarantine): [[Exchange Online Administration]].
