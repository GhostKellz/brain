---
type: reference
title: "acme.sh — DNS-01 Certificates"
created: 2026-06-22
updated: 2026-06-22
tags:
  - acme
  - letsencrypt
  - tls
  - dns
  - cloudflare
  - azure
status: developing
related:
  - "[[Let's Encrypt - Certbot|Let's Encrypt / Certbot]]"
  - "[[Nginx Reference]]"
  - "[[Linux Administration]]"
---

> [!key-insight] Generalized from field notes; host/client-specific values are placeholders.

[acme.sh](https://github.com/acmesh-official/acme.sh) is a pure-shell ACME
client. The reason to reach for it over certbot is **DNS-01 validation**:
the cert is proved by writing a TXT record, so no port 80 listener is needed and
**wildcards** (`*.example.com`) are issuable. This page covers the two DNS
providers used here — **Cloudflare** and **Azure DNS** — for both Let's Encrypt
and the option of other ACME CAs. For the HTTP-01 / nginx-plugin path see
[[Let's Encrypt - Certbot|Let's Encrypt / Certbot]].

## Install

```bash
curl https://get.acme.sh | sh -s email=<you>@<domain>
# installs to ~/.acme.sh, adds a daily cron renew, aliases `acme.sh`
source ~/.bashrc      # or re-login so the alias loads
```

> [!key-insight]
> acme.sh installs its own cron entry on install (`acme.sh --cron`). You do
> **not** need a separate systemd timer for renewal — issuing once wires up
> auto-renew for that cert.

acme.sh defaults to ZeroSSL. Pin Let's Encrypt as the default CA so issuance is
predictable:

```bash
acme.sh --set-default-ca --server letsencrypt
```

## Method A — Cloudflare DNS

Create a **scoped** API token in Cloudflare (My Profile → API Tokens) with
`Zone:DNS:Edit` + `Zone:Zone:Read`, restricted to the target zone. Export it,
then issue:

```bash
export CF_Token="<cloudflare-api-token>"
export CF_Account_ID="<cloudflare-account-id>"   # optional but recommended

# Single host
acme.sh --issue --dns dns_cf -d <fqdn>.<domain>

# Wildcard + apex in one cert
acme.sh --issue --dns dns_cf -d <domain> -d '*.<domain>'
```

acme.sh stores the credentials in `~/.acme.sh/account.conf` after the first run,
so subsequent renewals don't need the env vars re-exported.

> [!warning]
> Use a **scoped token** (single zone, DNS edit only). The legacy Global API Key
> works but grants account-wide access — never put that in an automation host.

## Method B — Azure DNS

For zones hosted in **Azure DNS**, validation writes the TXT record via an Azure
service principal. Create an app registration / service principal with the **DNS
Zone Contributor** role scoped to the DNS zone's resource group, then:

```bash
export AZUREDNS_SUBSCRIPTIONID="<subscription-id>"
export AZUREDNS_TENANTID="<tenant-id>"
export AZUREDNS_APPID="<app-registration-client-id>"
export AZUREDNS_CLIENTSECRET="<client-secret>"

acme.sh --issue --dns dns_azure -d <fqdn>.<domain>
acme.sh --issue --dns dns_azure -d <domain> -d '*.<domain>'
```

Managed-identity mode (when running on an Azure VM with a system-assigned
identity, no secret needed):

```bash
export AZUREDNS_MANAGEDIDENTITY="true"
acme.sh --issue --dns dns_azure -d <fqdn>.<domain>
```

> [!key-insight]
> The service principal only needs **DNS Zone Contributor** on the zone's
> resource group — not Owner, not subscription-wide. Scope it tightly; this
> credential lives on an unattended renewal host.

## Install the cert (don't symlink `~/.acme.sh`)

Never point a web server at the files in `~/.acme.sh` directly — acme.sh rotates
those on renewal. Use `--install-cert` to copy them to a stable path and fire a
**reloadcmd**:

```bash
acme.sh --install-cert -d <domain> \
  --key-file       /etc/nginx/certs/<domain>/privkey.pem \
  --fullchain-file /etc/nginx/certs/<domain>/fullchain.pem \
  --reloadcmd      "systemctl reload nginx"
```

On every successful renewal acme.sh re-copies the PEMs and re-runs the
`reloadcmd`, so nginx always serves the fresh chain.

## Renewal & operations

```bash
acme.sh --list                      # installed certs + next renew date
acme.sh --renew -d <domain> --force # force a renew now
acme.sh --cron                      # what the cron job runs (renew-if-due)
acme.sh --remove -d <domain>        # stop tracking a cert (files stay on disk)
acme.sh --upgrade                   # update acme.sh itself
```

Let's Encrypt certs are 90-day; acme.sh renews at ~60 days by default. DNS-01
renewals are non-disruptive (no port juggling), which is the main operational
win over standalone HTTP-01.

## Choosing between acme.sh and certbot

| | acme.sh (DNS-01) | certbot |
|---|---|---|
| Wildcards | Yes | Only via DNS plugins |
| Port 80 needed | No | Yes (standalone/webroot) |
| Dependencies | Pure shell | Python |
| Renewal | Built-in cron | systemd `certbot.timer` |
| Best for | Wildcards, no-inbound hosts, DNS-API zones | Single hosts with port 80 + nginx plugin |

## Related

- [[Let's Encrypt - Certbot|Let's Encrypt / Certbot]] — the HTTP-01 / nginx-plugin path
- [[Nginx Reference]] — where the issued PEMs get wired in
- [[Linux Administration]] — host/cron context
