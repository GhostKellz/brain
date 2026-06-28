---
type: reference
title: "SMTP2GO"
aliases:
  - "SMTP2Go"
  - "SMTP Relay"
  - "SMTP"
created: 2026-06-28
updated: 2026-06-28
tags:
  - email
  - smtp
  - relay
  - deliverability
  - reference
status: developing
related:
  - "[[Email DNS Security]]"
  - "[[Uptime Kuma]]"
  - "[[Self-Hosted Services]]"
  - "[[Networking Reference]]"
---

> [!key-insight] SMTP2GO is an **outbound relay** (a.k.a. smarthost) for
> transactional/notification mail — app forms, server cron mail, [[Uptime Kuma]]
> alerts, monitoring. You point any SMTP client at it instead of running a public
> MTA, and it handles deliverability (IP reputation, SPF/DKIM alignment). It is
> **send-only**; it is not a mailbox host (that's M365/Exchange).

The house relay for "I just need this box/app to send email reliably." Create an
SMTP user, point the client at `mail.smtp2go.com`, done. Verifying your own
sending domain is **optional** (you can send immediately) but strongly recommended
for deliverability — see [DNS](#dns--deliverability).

## Setup

1. **Create the account**, then **add a sending domain** (optional but
   recommended — improves deliverability and lets mail come *from* your domain
   instead of a shared one).
2. **Create an SMTP user** — *Settings → Users → SMTP Users → Add SMTP User*.
   This username + password **is** your SMTP credential set. SMTP2GO auth is
   per-SMTP-user, **not** your account login — generate one per app/host so you
   can rotate/revoke independently.
3. **Connect** any client to `mail.smtp2go.com` on a submission port (below) with
   that SMTP user's credentials and `AUTH LOGIN`/`PLAIN` over TLS.

> [!note] Domain **verification is not required to send** — you can relay right
> away with an SMTP user. Verify the domain (SPF/DKIM) when you care about inbox
> placement and sending as `you@yourdomain`.

## Connection details

- **Server:** `mail.smtp2go.com`

| Mode | Ports | Notes |
|------|-------|-------|
| **STARTTLS / plain** | **2525** (default), 587, 8025, 80, 25 | TLS via STARTTLS available on all of these |
| **SSL / TLS (implicit)** | **465**, 8465, 443 | TLS from the first byte |

> [!key-insight] Use **587** (STARTTLS) or **465** (implicit SSL) for normal hosts.
> Fall back to **2525** when an ISP/cloud provider blocks 25/587 outbound (very
> common on residential and some VPS networks). The high alt-ports (8025/8465)
> and 80/443 exist for environments that only allow web ports outbound.

## Client examples

### msmtp (sendmail drop-in for cron / scripts)

```ini
# /etc/msmtprc   (chmod 600; it holds the password)
defaults
auth           on
tls            on
tls_starttls   on
logfile        /var/log/msmtp.log

account        smtp2go
host           mail.smtp2go.com
port           587
from           noreply@example.com
user           <smtp-username>
password       <smtp-password>

account default : smtp2go
```

```bash
echo -e "Subject: test\n\nhello" | msmtp you@example.com
```

For implicit SSL instead, use `port 465` and `tls_starttls off`.

### Uptime Kuma email notification

In [[Uptime Kuma]] → *Settings → Notifications → Email (SMTP)*:

- Host `mail.smtp2go.com`, Port `587`, **Secure: STARTTLS** (or `465` + SSL/TLS)
- Username/Password = the SMTP user
- From = a verified-domain address for best deliverability

### Generic app / framework

Any `SMTP_HOST=mail.smtp2go.com`, `SMTP_PORT=587`, `SMTP_USER`/`SMTP_PASS`,
`SMTP_SECURE=starttls` (or 465/`ssl`) works the same way.

## DNS & deliverability

Verifying the sending domain publishes the records that stop your mail landing in
spam — full mechanics in [[Email DNS Security]]:

- **SPF** — `include:spf.smtp2go.net` in your domain's TXT SPF record.
- **DKIM** — add the CNAME(s) SMTP2GO shows for the domain; they sign outbound mail.
- **Return-Path / link-tracking** CNAMEs — optional, improve alignment + tracking.
- Optionally tighten **DMARC** once SPF+DKIM align ([[Email DNS Security]]).

## Test & debug

```bash
# swaks — the SMTP swiss-army knife
swaks --to you@example.com --from noreply@example.com \
  --server mail.smtp2go.com:587 --tls \
  --auth LOGIN --auth-user '<smtp-username>' --auth-password '<smtp-password>'

# Raw TLS handshake / banner check
openssl s_client -starttls smtp -connect mail.smtp2go.com:587 -crlf
openssl s_client -connect mail.smtp2go.com:465 -crlf      # implicit SSL
```

- **535 auth failed** → wrong SMTP-user creds (not your account login).
- **Connection times out on 25/587** → ISP/provider blocks it; switch to **2525**.
- **Mail sends but lands in spam** → SPF/DKIM not aligned; verify the domain
  ([[Email DNS Security]]).
- Check the **Activity / bounce logs** in the SMTP2GO dashboard for per-message fate.

## Notes

- Mailbox hosting (inbound, calendars, users) is M365/Exchange, not this →
  [[Microsoft 365 Administration]] · [[Exchange Online Administration]].
- DNS records for deliverability: [[Email DNS Security]].
- A common consumer: monitoring alerts → [[Uptime Kuma]].
