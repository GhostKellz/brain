---
type: reference
title: "UniFi Controller"
created: 2026-06-21
updated: 2026-06-21
tags:
  - unifi
  - networking
  - adoption
status: developing
related:
  - "[[Let's Encrypt - Certbot|Let's Encrypt / Certbot]]"
  - "[[FortiGate Administration]]"
  - "[[Networking Reference]]"
---

> [!key-insight] Generalized from field notes; host/client-specific values are placeholders.

Adopting UniFi devices to a self-hosted controller, including Layer 3 adoption patterns and TLS certs.

## Adopt a Device via `set-inform`

SSH into the device (default credentials or your provisioning credentials), then point it at the controller's inform endpoint. Replace `<controller-host>` with the controller's FQDN or IP.

```bash
set-inform https://<controller-host>:8080/inform
```

If you hit an `Unknown[12]` error, retry over plain HTTP first, then re-adopt:

```bash
set-inform http://<controller-host>:8080/inform
```

You can also use a controller IP directly:

```bash
set-inform http://<ip>:8080/inform
```

## Layer 3 Adoption (Remote Networks)

When devices and controller are on different subnets, use one of:

- **DHCP Option 43** — encode the controller IP as hex. See the FortiGate DHCP Option 43 recipe in [[FortiGate Administration]] and UI's L3 adoption docs.
- **Local DNS override** — create an `A` record named `unifi` pointing at the controller, so devices resolve the default `unifi` hostname to your controller:

```
unifi  ->  <controller-ip>
```

## TLS Certificate (Let's Encrypt)

Issue a standalone cert for the controller hostname, then wire it into the controller / reverse proxy. Stop the web server on port 80 first so certbot can bind it.

```bash
sudo systemctl stop nginx
sudo certbot certonly --standalone -d unifi.<domain>
sudo nano /etc/nginx/sites-available/unifi
```

Full standalone issuance and nginx integration details: [[Let's Encrypt - Certbot|Let's Encrypt / Certbot]].

## Notes

- The controller is commonly hosted on a Linux VM or LXC; see [[Proxmox Administration]].
