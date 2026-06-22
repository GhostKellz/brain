---
type: reference
title: "Let's Encrypt / Certbot"
created: 2026-06-21
updated: 2026-06-21
tags:
  - certbot
  - letsencrypt
  - tls
  - nginx
status: developing
related:
  - "[[UniFi Controller]]"
  - "[[Nginx Reference]]"
  - "[[Linux Administration]]"
---

> [!key-insight] Generalized from field notes; host/client-specific values are placeholders.

Issuing and renewing Let's Encrypt certificates with certbot (standalone and nginx plugin), and dropping the resulting cert into appliances.

## Install

```bash
sudo apt-get update
sudo apt-get install certbot python3-certbot-nginx   # nginx plugin
# or, minimal:
sudo apt install certbot -y
```

## Standalone Issuance

Certbot spins up a temporary listener on port 80, so free it first (stop nginx or any web server bound to 80):

```bash
sudo systemctl stop nginx
sudo certbot certonly --standalone -d <fqdn>.<domain>
```

Certs land in `/etc/letsencrypt/live/<fqdn>.<domain>/`:
- `fullchain.pem` — certificate + chain
- `privkey.pem` — private key

## Nginx Plugin (Automatic Config)

Let certbot edit the nginx vhost and reload for you:

```bash
sudo certbot --nginx -d <fqdn>.<domain>
```

## Deploy Cert to an Appliance

Many appliances (e.g. KASM, UniFi) want you to copy the PEMs in and restart the service. Generic pattern:

```bash
# stop service
sudo <appliance> stop

cp /etc/letsencrypt/live/<fqdn>.<domain>/fullchain.pem  <appliance-cert-path>.crt
cp /etc/letsencrypt/live/<fqdn>.<domain>/privkey.pem    <appliance-cert-path>.key

# start service
sudo <appliance> start
```

## Renewal

```bash
sudo certbot certificates            # list installed certs + expiry
sudo certbot renew                   # renew anything near expiry
sudo certbot renew --force-renewal -d <fqdn>.<domain>
```

Automate via the systemd `certbot.timer` (default) or a cron job. For appliances that need the cert copied in after renewal, add a `--deploy-hook` that re-runs the copy + restart steps above.

## Notes

- See [[UniFi Controller]] for controller-specific cert placement.
- See [[Nginx Reference]] for testing/reloading nginx after cert changes.
