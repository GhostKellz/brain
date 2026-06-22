---
type: reference
title: "Headscale"
created: 2026-06-22
updated: 2026-06-22
tags:
  - headscale
  - tailscale
  - vpn
  - wireguard
  - networking
  - zero-trust
  - self-hosting
status: developing
related:
  - "[[Tailscale]]"
  - "[[WireGuard]]"
  - "[[Mesh VPN]]"
  - "[[Zero Trust Networking]]"
  - "[[Self-Hosted Services]]"
  - "[[acme.sh - DNS-01 Certificates]]"
---

> [!key-insight] **Headscale** is an open-source, self-hosted reimplementation of
> Tailscale's **control plane** (the coordination server). You pair it with the
> *normal, unmodified* Tailscale **client** — point the client at your server
> with `--login-server`. The data plane (WireGuard tunnels, NAT traversal, DERP)
> is identical to SaaS; only the coordination/identity layer changes. See
> [[Tailscale]] for the client side.

> [!note] All hostnames/IPs here are placeholders (`headscale.example.com`,
> `100.x.y.z`). Real topology lives in the private tier.

## When to self-host

- You want the control plane **on your own infrastructure** (no third-party
  coordination server) for a lab, homelab, or a small org.
- Scope is deliberately narrow: Headscale targets a **single tailnet** for
  personal / small-org use — not multi-org enterprise.
- You're fine trading the polished admin console + managed DERP for a YAML file
  and a CLI.

## What it supports vs. SaaS

**Supported:** pre-auth keys, ACL policy file, tags, subnet routes & exit nodes,
MagicDNS, OIDC/OpenID Connect login, embedded **or** custom DERP, an API, and
(via third-party) a web UI.

**Gaps vs. Tailscale SaaS:** single-tailnet only (no multi-org/enterprise),
no first-party GUI admin console (CLI + community web UIs), no managed global
DERP fleet, and feature parity lags the SaaS product. Funnel and some
SaaS-only niceties aren't first-class.

## Install (Debian/Ubuntu, systemd)

The `.deb` creates a dedicated user, systemd unit, and config skeleton:

```bash
HEADSCALE_VERSION="X.Y.Z"   # latest release
HEADSCALE_ARCH="amd64"
wget -O headscale.deb \
  "https://github.com/juanfont/headscale/releases/download/v${HEADSCALE_VERSION}/headscale_${HEADSCALE_VERSION}_linux_${HEADSCALE_ARCH}.deb"
sudo apt install ./headscale.deb
sudo systemctl enable --now headscale
sudo systemctl status headscale
```

Config lives at **`/etc/headscale/config.yaml`** (example at
`/usr/share/doc/headscale/examples/config-example.yaml`); the CLI talks to the
daemon over `/var/run/headscale/headscale.sock`.

### Docker alternative

```bash
docker run -d --name headscale \
  -v /etc/headscale:/etc/headscale \
  -v /var/lib/headscale:/var/lib/headscale \
  -p 8080:8080 -p 3478:3478/udp \
  headscale/headscale:stable serve
# CLI then runs inside the container:
docker exec -it headscale headscale users list
```

## Key config (`/etc/headscale/config.yaml`)

```yaml
server_url: https://headscale.example.com    # what clients pass to --login-server
listen_addr: 0.0.0.0:8080                     # control-plane API/HTTP
metrics_listen_addr: 127.0.0.1:9090

database:
  type: sqlite                                # sqlite for small; postgres for bigger
  sqlite:
    path: /var/lib/headscale/db.sqlite

dns:
  base_domain: tailnet.example.com            # MagicDNS suffix (must differ from server_url host)
  magic_dns: true
  nameservers:
    global:
      - 1.1.1.1

derp:
  server:
    enabled: true                             # run an embedded DERP relay
    region_id: 999
    stun_listen_addr: 0.0.0.0:3478
  urls:                                        # or pull the public Tailscale DERP map
    - https://controlplane.tailscale.com/derpmap/default
```

`server_url` is the single most important value — it's exactly what clients use
as `--login-server`. `base_domain` must **not** be a subdomain of the
`server_url` host.

## Users, keys, and registering nodes

```bash
# create a user (namespace nodes attach to)
headscale users create alice

# --- option A: interactive/web registration ---
# on the client:
sudo tailscale up --login-server https://headscale.example.com
# it prints an Auth ID; approve it on the server:
headscale auth register --user alice --auth-id <AUTH_ID>

# --- option B: pre-auth key (containers/IaC/headless) ---
headscale preauthkeys create --user alice --reusable --expiration 24h
# on the client:
sudo tailscale up --login-server https://headscale.example.com --authkey <KEY>

# inspect
headscale users list
headscale nodes list
```

Pre-auth keys default to single-use, 1-hour expiry; add `--reusable` and
`--ephemeral` as needed (ephemeral nodes auto-deregister when offline).

## Routes, exit nodes, DNS

Advertise/accept routes exactly like SaaS on the client (see
[[Tailscale]]), but **approve** them with the Headscale CLI:

```bash
# client advertises:
sudo tailscale set --advertise-routes=192.0.2.0/24 --advertise-exit-node
# server approves:
headscale nodes list-routes
headscale nodes approve-routes --identifier <NODE_ID> --routes 192.0.2.0/24
```

MagicDNS comes from the `dns.base_domain` / `magic_dns` config above.

## ACL policy (zero trust)

Headscale reads the same **Tailscale-style policy file** (HuJSON) — `tagOwners`,
`groups`, `acls`, `ssh`. Point the config at it:

```yaml
policy:
  mode: file
  path: /etc/headscale/acl.hujson
```

```json
{
  "tagOwners": { "tag:server": ["alice@"] },
  "acls": [
    { "action": "accept", "src": ["alice@"], "dst": ["tag:server:22,443"] }
  ]
}
```

Reload after edits: `sudo systemctl reload headscale` (or `restart`). The
zero-trust model is identical to [[Tailscale]] — default-deny, grant
by identity/tag.

## DERP relays (self-hosted)

Two choices:
1. **Embedded DERP** — flip `derp.server.enabled: true` (config above); good for
   a single self-contained box. Expose its STUN port (`3478/udp`).
2. **Standalone `derper`** — run Tailscale's `cmd/derper` separately and list it
   in a custom DERP map. More moving parts; only needed at scale or for
   geo-spread relays.

Self-hosting DERP matters more here than on SaaS: there's no managed global
relay fleet, so a reachable DERP is your fallback when peers can't go direct.

## TLS / reverse proxy

`server_url` is HTTPS, so terminate TLS either with Headscale's built-in ACME or
(more commonly) a reverse proxy:

- **Reverse proxy** (nginx/Caddy) in front of `listen_addr` (`:8080`), with a
  Let's Encrypt cert for `headscale.example.com` — issue via DNS-01, see
  [[acme.sh - DNS-01 Certificates]]. Proxy WebSocket upgrades through for DERP.
- **Built-in ACME** — Headscale can fetch its own cert if exposed on 80/443.

## Limitations to remember

- One tailnet, small-scale focus; not an enterprise multi-org replacement.
- No official admin GUI — CLI plus community web UIs.
- Feature parity trails Tailscale SaaS; verify a feature exists before designing
  around it. For production identity/SSO at scale, SaaS Tailscale is usually the
  pragmatic choice — keep self-hosted Headscale to lab/homelab roles.

## Related

- [[Tailscale]] — the client side + control-plane vs data-plane overview.
- [[WireGuard]] · [[Mesh VPN]] · [[Zero Trust Networking]] — the fundamentals.
- [[acme.sh - DNS-01 Certificates]] — TLS for the `server_url` host.
- [[Self-Hosted Services]] — wider self-hosting context.
