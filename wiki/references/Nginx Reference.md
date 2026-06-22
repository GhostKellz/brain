---
type: reference
title: "Nginx Reference"
created: 2026-06-21
updated: 2026-06-21
tags:
  - nginx
  - reverse-proxy
  - linux
status: developing
related:
  - "[[Let's Encrypt - Certbot|Let's Encrypt / Certbot]]"
  - "[[Docker and Portainer]]"
---

> [!key-insight] Generalized from field notes; host/client-specific values are placeholders.

Common nginx operational commands for host-installed and containerized deployments.

## Host-installed

```bash
sudo nginx -t                      # validate config
sudo systemctl restart nginx       # restart
```

## Containerized

```bash
sudo docker exec <nginx-container> nginx -t          # validate config
sudo docker exec <nginx-container> nginx -s reload   # graceful reload
```

## Notes

- TLS certs and the standalone-then-reload pattern: [[Let's Encrypt - Certbot|Let's Encrypt / Certbot]].
- Running nginx in Docker: [[Docker and Portainer]].
