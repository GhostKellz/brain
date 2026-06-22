---
type: reference
title: "Self-Hosted Services"
created: 2026-06-21
updated: 2026-06-21
tags:
  - selfhosted
  - pihole
  - kasm
  - linux
status: developing
related:
  - "[[Let's Encrypt - Certbot|Let's Encrypt / Certbot]]"
  - "[[Docker and Portainer]]"
  - "[[Tailscale]]"
---

> [!key-insight] Generalized from field notes; host/client-specific values are placeholders.

Install/upgrade notes for commonly self-hosted services: Pi-hole and KASM Workspaces.

## Pi-hole

```bash
# Install
curl -sSL https://install.pi-hole.net | bash

# Update Pi-hole and gravity (blocklists)
pihole -up
pihole -g

# Change web admin password
sudo pihole -a -p

# Restart the FTL resolver
sudo systemctl restart pihole-FTL.service
```

> [!note] Pi-hole pairs well with Tailscale's `--accept-dns=false` so tailnet clients keep using your local resolver. See [[Tailscale]].

## KASM Workspaces

### Prep (swap recommended)

See the swap-file recipe in [[Linux Administration]].

### Install

```bash
wget <kasm-release-tarball-url>
tar -xf kasm_release*.tar.gz
sudo bash kasm_release/install.sh
```

### Replace self-signed cert with Let's Encrypt

Issue a standalone cert (see [[Let's Encrypt - Certbot|Let's Encrypt / Certbot]]), then swap KASM's nginx cert/key and restart:

```bash
sudo /opt/kasm/bin/stop
cp /etc/letsencrypt/live/<fqdn>.<domain>/fullchain.pem /opt/kasm/current/certs/kasm_nginx.crt
cp /etc/letsencrypt/live/<fqdn>.<domain>/privkey.pem   /opt/kasm/current/certs/kasm_nginx.key
sudo /opt/kasm/bin/start
```

Set up a cron/`--deploy-hook` so renewals re-copy the cert and restart KASM.

## Notes

- Container-based deployments: [[Docker and Portainer]].
