---
type: reference
title: "Cloudflare"
created: 2026-06-22
updated: 2026-06-22
tags:
  - cloudflare
  - dns
  - cdn
  - waf
  - zero-trust
  - security
  - networking
  - caching
status: developing
related:
  - "[[Zero Trust Networking]]"
  - "[[FortiGate Administration]]"
  - "[[acme.sh - DNS-01 Certificates]]"
  - "[[Networking Reference]]"
  - "[[Nginx Reference]]"
  - "[[Email DNS Security]]"
---

> [!key-insight] Cloudflare sits **in front of** your origin as an authoritative
> DNS provider + reverse proxy. The single most important toggle is the
> **orange-cloud vs grey-cloud** switch per DNS record: orange = traffic is
> **proxied** through Cloudflare (CDN, WAF, bot/AI blocking, hidden origin IP);
> grey = **DNS-only** (Cloudflare just answers DNS, traffic goes straight to your
> IP). Everything below only applies to **orange-cloud** records.

> [!note] All zones/hostnames/IPs here are placeholders (`example.com`,
> `203.0.113.x`). Real records live in the private tier.

## Orange cloud vs grey cloud

| | Orange (proxied) | Grey (DNS-only) |
|---|---|---|
| Origin IP | hidden behind Cloudflare | exposed in DNS |
| CDN / cache | yes | no |
| WAF / firewall rules | yes | no |
| Bot / AI-scraper blocking | yes | no |
| Universal SSL (edge cert) | yes | no |
| Used here for | public web apps | `brain.ckelley.dev` (self-hosted TLS on origin) |

> [!note] `brain.ckelley.dev` is intentionally **grey-cloud** — the origin
> (thallium) terminates its own Let's Encrypt cert via [[acme.sh - DNS-01
> Certificates]], and the FortiGate geo-allowlist does the gating. Orange-cloud
> would hide the origin and add the WAF, at the cost of Cloudflare seeing the
> traffic.

## Blocking AI scrapers (orange cloud)

The headline feature for a public knowledge site. Cloudflare maintains a managed
rule that blocks known AI crawlers — you toggle it, you don't author it.

- **One-click block:** Dashboard → **Security → Bots → "Block AI bots"** (a.k.a.
  *AI Scrapers and Crawlers*). Free on all plans, auto-updated as Cloudflare
  fingerprints new scrapers.
- **What it blocks:** verified AI crawlers (GPTBot, ClaudeBot, CCBot,
  Meta-ExternalAgent, Bytespider, Amazonbot, …) plus unverified bots behaving
  like them. **Search engines** (Googlebot/Bingbot) are *not* affected.
- **AI Crawl Control** (formerly *AI Audit*): dashboard view of which AI services
  hit your content, per-crawler allow/block policies, robots.txt-compliance
  tracking, and an optional **pay-per-crawl** monetization path. Blocks here are
  enforced as a **WAF custom rule** on the zone, which you can extend (e.g.
  path-based exceptions, allow a partner bot).
- **robots.txt is honor-system only** — bots can ignore it, which is exactly why
  the managed WAF rule exists. Use both: robots.txt as the polite signal, the
  managed rule as enforcement.

Evaluation order: a Bot Management custom rule runs **before** the Block-AI-bots
rule, which runs before the rest of Super Bot Fight Mode.

## WAF / firewall rules

The Web Application Firewall is the policy engine for proxied traffic:

- **Managed rulesets** — Cloudflare-maintained OWASP-style protections; toggle +
  tune sensitivity.
- **Custom rules** — expression-based (Wireshark-like syntax) to
  `block` / `challenge` / `skip` / `log`:

  ```text
  # block everything except an allowlisted country (mirrors the FortiGate geo-allowlist)
  (not ip.geoip.country in {"US" "CA" "GB"})

  # rate-limit or challenge an abusive path
  (http.request.uri.path contains "/wp-login")
  ```

- **Rate limiting rules** — per-IP/path request thresholds.
- **IP Access rules / ACLs** — allow/block/challenge by IP, CIDR, ASN, or
  country (the simple ACL layer beneath custom rules).

This complements an edge firewall like [[FortiGate Administration]]: Cloudflare
filters at the global edge, FortiGate enforces at your WAN.

## Caching

- **Default cache** keys on static file extensions; HTML is *not* cached by
  default.
- **Cache Rules** (the modern replacement for Page Rules) — match by hostname /
  path / extension and set **Cache Eligibility**, **Edge TTL**, **Browser TTL**,
  and cache-key behavior.
- **Cache Everything** + an Edge TTL turns Cloudflare into a full-page cache for
  a static site (e.g. a Quartz/Astro build) — good fit for content that changes
  on a publish cadence.
- **Tiered Cache / Argo** — upper-tier data centers reduce origin fetches.
- **Purge** — by URL, tag, hostname, or everything, after a deploy.

## Zero Trust (Cloudflare One)

Cloudflare's [[Zero Trust Networking]] suite — identity-gated access without a
classic VPN:

- **Access** — put SSO (OIDC/SAML) in front of a self-hosted app; policies by
  identity, group, device posture, country. Replaces a VPN for app exposure.
- **Tunnel (`cloudflared`)** — outbound-only daemon that connects an origin to
  the edge, so you publish an app **without opening inbound ports** or exposing
  the origin IP. Conceptually similar to `tailscale funnel` (see
  [[Tailscale Operations]]) but on Cloudflare's edge.
- **Gateway** — egress/DNS/HTTP filtering policies for outbound traffic
  (content categories, security, **egress policies**), the SWG half of the
  story.
- **WARP** — the client that puts devices onto the Zero Trust network.

```bash
# expose a local service via Tunnel, no inbound firewall holes
cloudflared tunnel login
cloudflared tunnel create example
cloudflared tunnel route dns example app.example.com
cloudflared tunnel run example      # ingress mapped in config.yml → http://localhost:3000
```

## Traffic / routing & egress

- **Load Balancing** — health-checked pools, steering by geo/latency/weight.
- **Origin Rules** — rewrite host header, override origin, change ports per match.
- **Transform / Redirect Rules** — header/URL rewrites and bulk redirects at the
  edge.
- **Gateway egress policies** — control how outbound traffic leaves (dedicated
  egress IPs, per-policy routing) for the Zero Trust side.

## TLS / certificates

- **Universal SSL** — free edge cert for proxied (orange) hostnames, auto-renewed.
- **Origin CA cert** — long-lived cert for the Cloudflare↔origin hop (trusted
  only by Cloudflare).
- **SSL/TLS mode** — use **Full (strict)** so the origin must present a valid
  cert; never use **Flexible** (plaintext to origin).
- For **grey-cloud** hosts Cloudflare issues **no** cert — the origin owns TLS;
  here that's Let's Encrypt **DNS-01** via the Cloudflare API token, see
  [[acme.sh - DNS-01 Certificates]].

## DNS notes

- Authoritative DNS with per-record proxy toggle; very fast propagation.
- A **scoped API token** (Zone:DNS:Edit on one zone) is the right credential for
  ACME DNS-01 automation — never a Global API Key.
- Email auth records (SPF/DKIM/DMARC) live here too — see [[Email DNS Security]].

## Related

- [[Zero Trust Networking]] — the model behind Access/Gateway/Tunnel.
- [[FortiGate Administration]] — the WAN-edge firewall pairing.
- [[acme.sh - DNS-01 Certificates]] — DNS-01 with a scoped Cloudflare token.
- [[Nginx Reference]] · [[Networking Reference]] · [[Email DNS Security]].
