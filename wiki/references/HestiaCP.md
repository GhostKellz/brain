---
type: reference
title: "HestiaCP"
created: 2026-06-22
updated: 2026-06-22
tags:
  - hestiacp
  - hosting
  - control-panel
  - nginx
  - dns
  - mail
  - self-hosting
status: developing
related:
  - "[[Self-Hosted Services]]"
  - "[[GhostCP]]"
  - "[[Nginx Reference]]"
  - "[[acme.sh - DNS-01 Certificates]]"
  - "[[Email DNS Security]]"
  - "[[Let's Encrypt - Certbot|Let's Encrypt / Certbot]]"
---

> [!key-insight] **HestiaCP** (Hestia Control Panel) is an open-source web
> hosting control panel — a single pane that templates and drives the native
> Linux hosting stack (web, DNS, mail, DB, FTP, firewall) so you manage domains,
> mailboxes, and certs from a UI or the `v-` CLI instead of editing each
> service's config by hand. GPLv3; a community fork of VestaCP.

> [!note] All hostnames/users/domains here are placeholders
> (`panel.example.com`, `admin`, `example.com`). Real values live in the private
> tier.

## What it manages

HestiaCP doesn't replace the services — it **templates** them and keeps them in
sync with its own system-of-record:

| Function | Service it drives |
|---|---|
| Web (reverse proxy) | **nginx** in front of |
| Web (backend) | **Apache** + **PHP-FPM** (multi-PHP) — or nginx-only with PHP-FPM |
| DNS | **Bind9** (authoritative, with clustering) |
| Mail (MTA) | **Exim** + **ClamAV** + **SpamAssassin** |
| Mail (IMAP/POP) | **Dovecot** + Sieve |
| Database | **MariaDB/MySQL** and/or **PostgreSQL** |
| FTP | **vsftpd** (or ProFTPD) |
| Firewall / IPS | **iptables** + **Fail2ban** |
| TLS | **Let's Encrypt** (ACME, auto-renew) |

The default web model is **nginx (proxy) → Apache (backend)**; you can also run
**nginx + PHP-FPM only** (lighter, no Apache) per-template.

## Install

One bootstrap script, run as **root** on a clean supported OS (Debian / Ubuntu).
Generate the exact command from the official installer page so the flags match
your stack choices:

```bash
wget https://raw.githubusercontent.com/hestiacp/hestiacp/release/install/hst-install.sh
sudo bash hst-install.sh \
  --interactive no \
  --hostname panel.example.com \
  --email admin@example.com \
  --password '<set-strong>' \
  --apache yes --phpfpm yes \
  --named yes --exim yes --dovecot yes \
  --clamav yes --spamassassin yes \
  --mysql yes --postgresql no \
  --vsftpd yes --iptables yes --fail2ban yes
```

- Admin UI then lives at **`https://panel.example.com:8083`** (the panel binds a
  high port, separate from hosted sites on 80/443).
- First login is the `admin` superuser; create unprivileged **users** for actual
  hosting so a compromise of one account doesn't own the box.

## Users, packages, domains

HestiaCP is **multi-tenant**: each *user* owns their web/mail/DNS/DB, and a
*package* caps their quotas (disk, bandwidth, # of domains/DBs/mailboxes).

```bash
# packages (quota templates) and users
v-list-user-packages
v-add-user alice 'StrongPass' alice@example.com default 'Alice'

# web domain (+ optional aliases), then issue Let's Encrypt for it
v-add-web-domain alice example.com
v-add-letsencrypt-domain alice example.com 'www.example.com'

# dns zone + a record
v-add-dns-domain alice example.com 203.0.113.10
v-add-dns-record alice example.com www A 203.0.113.10

# mail domain + mailbox
v-add-mail-domain alice example.com
v-add-mail-account alice example.com bob 'MailboxPass'

# database
v-add-database alice app appuser 'DbPass' mysql
```

Everything the UI does maps to a `v-*` command in `/usr/local/hestia/bin/`
(700+ of them: `v-add-*`, `v-delete-*`, `v-list-*`, `v-change-*`). That makes
HestiaCP scriptable — provision a whole tenant from a shell loop or config-mgmt.

## TLS / Let's Encrypt

- Built-in **ACME (Let's Encrypt)** per web/mail domain via
  `v-add-letsencrypt-domain`; auto-renews on a cron.
- The panel's **own** hostname cert is managed with `v-add-letsencrypt-host`.
- HTTP-01 is the default. For **wildcard** or split-horizon hosts you want
  **DNS-01** instead — pair with [[acme.sh - DNS-01 Certificates]] (e.g. a
  scoped Cloudflare token) rather than the built-in HTTP-01 flow.
- Mail TLS (Exim/Dovecot) and FTPS reuse the issued certs.

## Mail & DNS hardening

- HestiaCP can emit **SPF/DKIM** records when you add a mail domain; **DMARC**
  you add/tune yourself. See [[Email DNS Security]] for the policy side.
- DKIM keys are generated per mail domain; publish the selector TXT in the zone
  (HestiaCP's Bind manages it if it's authoritative for that domain).
- Lock down the panel: restrict `:8083` to a mgmt network/VPN (e.g. a
  [[Tailscale]] tailnet) rather than the open internet.

## Security: Fail2ban + firewall

- **iptables** rules and **Fail2ban** jails ship enabled for SSH, the panel,
  mail (Exim/Dovecot), and FTP — brute-force lockouts out of the box.
- Manage from the UI (Firewall / Banned IPs) or:

```bash
v-list-firewall
v-add-firewall-rule ACCEPT 203.0.113.0/24 8083 TCP 'panel mgmt net'
v-list-firewall-ban
```

- Pairs with an upstream edge firewall ([[FortiGate Administration]]) and/or a
  CDN/WAF ([[Cloudflare]]) for defense in depth — keep the origin minimal.

## Backups

- Per-user backups bundle web + DB + mail + config into one archive:

```bash
v-backup-user alice                 # create
v-list-user-backups alice
v-restore-user alice alice.<ts>.tar # restore
```

- Local by default; configure **remote** backup targets (SFTP/FTP, or an S3-style
  endpoint via the community option) so the archive leaves the host. Fits a
  [[3-2-1 Backup Strategy]].

## Updates

- Auto-updates are **on by default** (apt + a HestiaCP package channel); the
  panel pulls new templates/binaries on a schedule.

```bash
v-list-sys-hestia-updates
v-update-sys-hestia          # pull the panel itself
v-update-sys-hestia-all      # panel + templated configs
```

## Where it fits here

HestiaCP is the **off-the-shelf** control panel; [[GhostCP]] is the Rust
reimagining of the same idea (Axum + Leptos + Postgres, standards-based DNS)
for when the templated-PHP-stack model isn't the goal. For ad-hoc single
services without a panel, see [[Self-Hosted Services]].

## Related

- [[GhostCP]] — the Rust "HestiaCP reimagined" control plane.
- [[Self-Hosted Services]] — panel-less self-hosting notes.
- [[Nginx Reference]] — the web tier HestiaCP templates.
- [[acme.sh - DNS-01 Certificates]] · [[Let's Encrypt - Certbot|Let's Encrypt / Certbot]] — TLS issuance.
- [[Email DNS Security]] — SPF/DKIM/DMARC for the mail side.
