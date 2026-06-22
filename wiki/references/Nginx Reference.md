---
type: reference
title: "Nginx Reference"
created: 2026-06-21
updated: 2026-06-22
tags:
  - nginx
  - reverse-proxy
  - webserver
  - tls
  - linux
status: developing
related:
  - "[[Let's Encrypt - Certbot|Let's Encrypt / Certbot]]"
  - "[[acme.sh - DNS-01 Certificates]]"
  - "[[NGINX Static Site Hosting]]"
  - "[[Self-Hosted Services]]"
  - "[[Docker and Portainer]]"
  - "[[FortiGate Administration]]"
  - "[[Cloudflare]]"
---

> [!key-insight] nginx is configured top-down: one `http` block contains many
> `server` blocks (virtual hosts), each selected by **`listen` + `server_name`**
> (and SNI for TLS), and each `server` contains `location` blocks matched against
> the request URI. Get a request to the right `server`/`location` and the rest is
> just directives. Generalized from field notes; all hostnames/paths/IPs are
> placeholders (`example.com`, `/var/www/site`, `203.0.113.x`).

## CLI & service control (host-installed)

```bash
sudo nginx -t                 # validate config syntax
sudo nginx -T                 # validate AND dump the full MERGED config, then exit
sudo nginx -s reload          # graceful reload (re-read config, no dropped conns)
sudo nginx -s reopen          # reopen log files (after logrotate)
sudo nginx -s quit            # graceful stop (finish in-flight requests)
sudo nginx -s stop            # fast shutdown
sudo nginx -V                 # version + compiled-in modules + ./configure flags (stderr)
sudo nginx -c /path/nginx.conf  # run with an alternate config file
sudo nginx -g 'daemon off;'   # set a global directive (foreground — common in containers)
```

Via systemd (preferred on a host):

```bash
sudo systemctl reload nginx     # graceful — sends HUP, keeps connections; use after edits
sudo systemctl restart nginx    # full restart — drops connections; only if reload won't do
sudo systemctl status nginx
journalctl -u nginx -e          # service-level errors (startup/permission failures)
```

> [!note] **`reload` over `restart`.** `nginx -t` then `systemctl reload` is the
> safe edit loop — a bad config fails `-t` and the running server keeps serving.
> `restart` briefly drops listeners.

## Containerized

```bash
sudo docker exec <nginx-container> nginx -t          # validate config
sudo docker exec <nginx-container> nginx -s reload   # graceful reload
sudo docker exec <nginx-container> nginx -T          # dump merged config from inside
```

## Config layout

Two common conventions — know which one the distro uses:

| Distro family | Entry | Vhost dir | Enable a site |
|---|---|---|---|
| RHEL / Arch | `/etc/nginx/nginx.conf` | `/etc/nginx/conf.d/*.conf` | drop a `.conf` in `conf.d/` |
| Debian / Ubuntu | `/etc/nginx/nginx.conf` | `sites-available/` | `ln -s` into `sites-enabled/` |

```bash
# Debian: enable / disable a vhost by symlink, then reload
sudo ln -s /etc/nginx/sites-available/site.conf /etc/nginx/sites-enabled/
sudo rm   /etc/nginx/sites-enabled/site.conf
sudo nginx -t && sudo systemctl reload nginx
```

- `nginx.conf` pulls vhosts in via `include /etc/nginx/conf.d/*.conf;` (and/or
  `sites-enabled/*`). The order of `include`s + `server_name` matching decides
  which block answers.
- Logs default to `/var/log/nginx/access.log` and `error.log` (override per
  `server` with `access_log` / `error_log`).

## Static site server block

The bread-and-butter vhost for a static build (Astro/Quartz/plain HTML):

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name example.com www.example.com;
    root /var/www/site;
    index index.html;

    location / {
        try_files $uri $uri/ $uri.html =404;   # serve file, dir, or .html; else 404
    }

    # long-cache fingerprinted assets
    location ~* \.(css|js|woff2|png|jpg|svg|webp)$ {
        expires 30d;
        add_header Cache-Control "public, immutable";
    }
}
```

`try_files` is the core static-serving directive: try each target left→right,
fall through to the last (`=404` or a named location). See
[[NGINX Static Site Hosting]] for the static-deploy workflow.

## Reverse proxy

Front an app server (Node, uvicorn, etc.) and pass real client info upstream:

```nginx
upstream app {
    server 127.0.0.1:3000;        # one or more backends
    # server 127.0.0.1:3001;      # add for round-robin
    keepalive 32;
}

server {
    listen 80;
    server_name app.example.com;

    location / {
        proxy_pass http://app;
        proxy_http_version 1.1;

        proxy_set_header Host              $host;
        proxy_set_header X-Real-IP         $remote_addr;
        proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # WebSocket upgrade (Socket.IO, HMR, etc.)
        proxy_set_header Upgrade    $http_upgrade;
        proxy_set_header Connection "upgrade";

        proxy_read_timeout 60s;
        proxy_connect_timeout 5s;
    }
}
```

- **`Host $host`** preserves the requested name so the backend routes vhosts.
- **`X-Forwarded-Proto $scheme`** lets the app know it's really HTTPS (for
  redirects/cookies) when TLS terminates here.
- Omit the WebSocket headers and live-reload/WS backends break — that's the most
  common "works on curl, fails in browser" proxy bug.

## TLS / HTTPS

Terminate TLS and force HTTP→HTTPS. Certs come from
[[Let's Encrypt - Certbot|Let's Encrypt / Certbot]] (HTTP-01) or
[[acme.sh - DNS-01 Certificates]] (DNS-01, wildcards):

```nginx
# redirect all plain HTTP to HTTPS
server {
    listen 80;
    listen [::]:80;
    server_name example.com www.example.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    listen [::]:443 ssl;
    http2 on;                        # nginx >= 1.25.1 (older: `listen 443 ssl http2;`)
    server_name example.com www.example.com;

    ssl_certificate     /etc/nginx/certs/example.com/fullchain.pem;
    ssl_certificate_key /etc/nginx/certs/example.com/privkey.pem;

    ssl_protocols       TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers off;
    ssl_session_cache   shared:SSL:10m;

    add_header Strict-Transport-Security "max-age=63072000; includeSubDomains" always;

    root /var/www/site;
    index index.html;
    location / { try_files $uri $uri/ $uri.html =404; }
}
```

- Multiple `443` vhosts on one IP are selected by **SNI** (`server_name`) — no
  need for separate IPs.
- After a cert renewal, `nginx -s reload` (or a certbot/acme.sh deploy hook) so
  the new cert is picked up without a restart.

## Hardening snippets

```nginx
# gzip (put in http{} or a server{})
gzip on;
gzip_types text/css application/javascript application/json image/svg+xml;
gzip_min_length 1024;

# baseline security headers
add_header X-Content-Type-Options nosniff always;
add_header X-Frame-Options SAMEORIGIN always;
add_header Referrer-Policy strict-origin-when-cross-origin always;

# rate limit a sensitive path (declare zone in http{}, apply in location{})
# http {  limit_req_zone $binary_remote_addr zone=login:10m rate=5r/s;  }
location /login {
    limit_req zone=login burst=10 nodelay;
    proxy_pass http://app;
}

# IP allow/deny (mirror an upstream geo/edge ACL — e.g. FortiGate / Cloudflare)
location /admin {
    allow 203.0.113.0/24;
    deny  all;
}
```

Pair host-level `allow/deny` with the edge: [[FortiGate Administration]] at the
WAN and/or [[Cloudflare]] WAF rules at the global edge.

## Debugging configs

```bash
# Which server block / file owns a server_name? Dump merged config and search:
sudo nginx -T | grep -n "server_name\|root\|listen"

# Test a specific vhost without DNS (force the Host header / SNI):
curl -ksI https://127.0.0.1 -H 'Host: example.com'      # local TLS vhost
curl -sI  http://127.0.0.1  -H 'Host: example.com'      # local HTTP vhost

# What is nginx actually listening on?
sudo ss -tlnp | grep nginx

# Live error tail while reproducing a request:
sudo tail -f /var/log/nginx/error.log
```

- `nginx -T` is the single best debugging tool — it shows the **effective**
  config after all `include`s, so duplicate `server_name`, wrong `root`, or a
  vhost that never got enabled all become obvious.
- A 404 on a static site is usually `root`/`try_files`; a 502/504 is the proxy
  backend down or a timeout; a "default" page served for the wrong host is a
  missing/duplicate `server_name`.

## Related

- [[NGINX Static Site Hosting]] — the static-deploy workflow this serves.
- [[Let's Encrypt - Certbot|Let's Encrypt / Certbot]] · [[acme.sh - DNS-01 Certificates]] — TLS issuance + renewal hooks.
- [[Self-Hosted Services]] — services that sit behind this proxy.
- [[Docker and Portainer]] — running nginx in a container.
- [[FortiGate Administration]] · [[Cloudflare]] — edge layers in front of nginx.
