---
type: concept
title: "NGINX Static Site Hosting"
complexity: beginner
domain: web
aliases:
  - NGINX static hosting
created: 2026-06-21
updated: 2026-06-21
tags:
  - concept
  - web
  - nginx
  - deployment
status: seed
related:
  - "[[Zero-JS Static Site Approach]]"
  - "[[Astro Static Site Generator]]"
  - "[[ckelley.dev]]"
  - "[[cktechx.com]]"
  - "[[ghostkellz.sh]]"
---

# NGINX Static Site Hosting

## Definition

All three of [[GhostKellz]]'s web properties are deployed the same way: build the
Astro site to a static `dist/` directory and serve those flat files with NGINX over
HTTPS. There is no application runtime — NGINX simply returns files.

## Deployment Pattern

The repos document a consistent build-and-sync flow:

```bash
pnpm build                                  # emit static output to dist/
rsync -avz --delete dist/ user@host:/var/www/<site>/
```

- **[[ckelley.dev]]** is served out of a web root such as `/var/www/ckelley.dev`.
- **[[cktechx.com]]** ships an NGINX config in `deploy/cktechx.conf`; after upload it
  is validated and reloaded with `nginx -t && systemctl reload nginx`. It serves both
  the main site and the `help.cktechx.com` subdomain, with TLS certs referenced from
  the server config.
- **[[ghostkellz.sh]]** deploys its `dist/` to the static host serving the domain.

> [!key-insight]
> Because the sites follow the [[Zero-JS Static Site Approach]], hosting is reduced to
> "serve files." That makes deploys atomic-ish (`rsync --delete`), rollbacks trivial,
> and the operational surface tiny — the web server is essentially the entire backend.

## Connections

- The deployment target for [[Astro Static Site Generator]] output across all three
  sites.
- A direct consequence of the [[Zero-JS Static Site Approach]].

## Sources

- [[ckelley.dev Repository]]
- [[cktechx.com Repository]]
- [[ghostkellz.sh Repository]]
