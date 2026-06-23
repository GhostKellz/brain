---
type: reference
title: "Tailscale"
aliases:
  - "Tailscale"
  - "entities/Tailscale"
  - "references/Tailscale Operations"
created: 2026-06-21
updated: 2026-06-22
tags:
  - tailscale
  - vpn
  - wireguard
  - networking
  - zero-trust
  - magicdns
  - mesh
status: developing
related:
  - "[[Headscale]]"
  - "[[Mesh VPN]]"
  - "[[WireGuard]]"
  - "[[Zero Trust Networking]]"
  - "[[FortiGate Administration]]"
  - "[[Linux Administration]]"
  - "[[Networking Reference]]"
---

# Tailscale

**Tailscale** is a [[Mesh VPN]] built on [[WireGuard]] with a managed control
plane. It creates a private overlay network (a "tailnet") where every enrolled
device gets a stable address and reaches every other device through direct,
encrypted peer-to-peer tunnels — without manual key management or port
forwarding.

> [!key-insight] Tailscale splits a VPN into a **control plane** (a coordination
> server that distributes WireGuard public keys + ACLs but never sees traffic)
> and a **data plane** (direct, end-to-end-encrypted WireGuard tunnels between
> peers). Its value is everything *around* WireGuard: identity (SSO/OIDC),
> automatic key distribution + rotation, NAT traversal, ACLs, and relay
> fallback. WireGuard is the tunnel; Tailscale is the coordination. Everything
> below — SSH, subnet routes, MagicDNS, serve/funnel, ACLs — builds on those two
> planes. To run the control plane yourself, see [[Headscale]].

> [!note] Generalized from field notes. All tailnet names, hostnames, and IPs
> here are placeholders (`tailnet-name.ts.net`, `100.x.y.z`, RFC-5737 example
> subnets). Real topology lives in the private tier.

## What it provides

- **Coordination server** — distributes keys and ACLs; never sees your traffic.
- **NAT traversal** — punches direct paths through most NATs/firewalls.
- **DERP relays** — fallback path when a direct connection can't be established;
  most traffic never touches one.
- **MagicDNS** — names instead of IPs across the tailnet.
- **ACLs** — per-identity access policy enabling [[Zero Trust Networking]].
- **SSO/OIDC login** — tie tailnet membership to an identity provider.

## How it works (one paragraph)

A coordination server distributes each node's WireGuard **public** key and the
tailnet ACL policy; private keys never leave the device. Nodes then use NAT
traversal to build **direct** peer-to-peer tunnels. When a direct path can't be
punched, traffic falls back to a [DERP relay](#derp-relays) — still end-to-end
encrypted, because the relay only forwards already-encrypted WireGuard packets.

## Install (Linux one-liner)

```bash
curl -fsSL https://tailscale.com/install.sh | sh     # official installer
sudo tailscale up                                    # auth in browser
sudo tailscale set --operator=$USER                  # let your user drive the CLI / GUIs
```

`tailscaled` (the daemon) is enabled as a systemd service by the installer:

```bash
sudo systemctl enable --now tailscaled
sudo systemctl status tailscaled
```

## Daily CLI

| Command | Purpose |
|---------|---------|
| `tailscale up` / `down` | connect (auth if needed) / disconnect |
| `tailscale set …` | change a single pref without re-running full `up` |
| `tailscale status` | peers, connection type (direct vs relay), routes |
| `tailscale ip -4` | this node's tailnet IP (100.x range) |
| `tailscale ping <host>` | ping **over Tailscale**; shows direct vs DERP |
| `tailscale netcheck` | NAT type, DERP latency, port-mapping report |
| `tailscale ssh <host>` | Tailscale-authenticated SSH (see below) |
| `tailscale serve` / `funnel` | expose a local service to tailnet / internet |
| `tailscale cert` | provision a Let's Encrypt cert for the MagicDNS name |
| `tailscale whois <ip>` | machine + user behind a tailnet IP |
| `tailscale file cp` | Taildrop file transfer |
| `tailscale lock` | manage tailnet lock |
| `tailscale update` | upgrade the client |

`tailscale netcheck` and `tailscale status` are the two go-to diagnostics: if a
peer shows `relay "xxx"` instead of `direct`, you're on DERP — see
[Fortinet](#firewalls--fortinet-cross-reference).

## tailscaled (the daemon)

The CLI is a thin client; `tailscaled` does the actual WireGuard/network work
and must be running. Useful flags (set in the systemd unit / drop-in):

```bash
--port=N                 # UDP port for peer traffic (default 41641; see Fortinet)
--tun=NAME               # TUN device name (or "userspace-networking")
--state=PATH             # state/key store (file, mem:, kube:<secret>, etc.)
--socket=PATH            # control socket the CLI talks to
--userspace-networking   # no kernel TUN — for containers/locked-down hosts
```

`--state=mem:` makes an ephemeral node (no persisted identity) — handy for
containers paired with an [ephemeral auth key](#auth-keys).

## Version & updates

```bash
tailscale version
tailscale update                    # latest in current track
tailscale update --version=1.80.0   # pin a version
tailscale set --auto-update         # enable client auto-update
```

## Subnet routing

A **subnet router** is a tailnet node that advertises routes to a physical
LAN, so tailnet peers can reach devices that *can't* run Tailscale (printers,
switches, IPMI, legacy servers).

```bash
# On the router node — advertise the LAN(s) it fronts:
sudo tailscale set --advertise-routes=192.0.2.0/24,198.51.100.0/24
# enable IP forwarding first (Linux):
echo 'net.ipv4.ip_forward = 1'  | sudo tee /etc/sysctl.d/99-tailscale.conf
echo 'net.ipv6.conf.all.forwarding = 1' | sudo tee -a /etc/sysctl.d/99-tailscale.conf
sudo sysctl -p /etc/sysctl.d/99-tailscale.conf
```

Then **approve** the routes in the admin console (Machines → the node → Subnets
→ Edit). On clients that should use them:

```bash
sudo tailscale set --accept-routes      # Linux must opt in; mobile/desktop auto-accept
```

Caveats (verified):
- **SNAT is on by default** — LAN devices see traffic as coming *from the
  router*. Disable with `--snat-subnet-routes=false` (Linux) only if you add
  return routes back to the tailnet.
- **Overlapping routes** resolve by longest-prefix match.
- For joining two LANs, prefer **site-to-site** networking over stacking subnet
  routers.

### Subnet router as a VPN replacement (remote access)

This is the "kill the legacy client VPN" pattern: stand up one subnet router on
the office/home LAN advertising the internal range, and remote staff reach
internal resources over the tailnet — no concentrator, no port-forwarding, no
shared VPN gateway. Combine with an **exit node** if you also want to route a
device's *internet* egress through a site:

```bash
sudo tailscale set --advertise-exit-node     # on the gateway node
sudo tailscale set --exit-node=<node-ip> --exit-node-allow-lan-access   # on the client
```

## DERP relays

DERP (Designated Encrypted Relay for Packets) servers do two jobs: help
**negotiate** direct connections, and act as a **fallback** path when no direct
tunnel can be built. Key facts:

- Traffic over DERP is still **WireGuard-encrypted end-to-end** — relays can't
  read it.
- Each client picks a **home DERP** by latency; Tailscale runs 20+ regions with
  automatic failover.
- You can self-host with `cmd/derper` and pin it via a `derpMap` in the policy
  file (mainly relevant to [[Headscale]]); regions can be disabled
  by setting their `RegionID` to `null`.
- Persistent DERP usage usually means a NAT/firewall problem — see Fortinet.

## Tailscale SSH (how it works)

Tailscale SSH reuses the WireGuard node identity instead of SSH key pairs. Enable
on the **destination** host:

```bash
sudo tailscale set --ssh
```

The node generates a host key, publishes the public half via the control plane,
and intercepts tailnet traffic to port 22. Because the tunnel already proves who
the peer is, the SSH auth phase is satisfied by **tailnet identity** — no
`authorized_keys` to manage. Access is governed by `ssh` rules in the policy
file:

```json
{
  "ssh": [
    {
      "action": "check",                       // "accept" = silent; "check" = re-auth via IdP
      "src":    ["autogroup:member"],
      "dst":    ["tag:server"],
      "users":  ["autogroup:nonroot", "root"],
      "checkPeriod": "12h"                      // re-auth window for "check"
    }
  ]
}
```

`action: "check"` forces the user to re-authenticate with the identity provider
before the session (default window 12h) — cheap just-in-time hardening for
sensitive hosts.

## MagicDNS

MagicDNS auto-registers a DNS name per device as
`machine-name.tailnet-name.ts.net`, so you address peers by name instead of
100.x IPs. Enable on the admin console DNS page (on by default for newer
tailnets). With **search domains** on, the bare `machine-name` resolves too.

```bash
sudo tailscale set --accept-dns=true     # honor tailnet DNS config (default)
sudo tailscale set --accept-dns=false    # keep the node on its local resolver
```

> [!note] `--accept-dns=false` is how you keep a node pointed at a **local
> resolver** (e.g. self-hosted Pi-hole / [[Unbound]]) while still on the tailnet.

## TLS certificates (how it works)

Tailscale provisions **real Let's Encrypt certs** for your MagicDNS name. Needs
MagicDNS + HTTPS enabled in the admin console (DNS page).

```bash
sudo tailscale cert machine-name.tailnet-name.ts.net
```

Mechanics: the daemon completes an **ACME DNS-01** challenge by publishing a TXT
record under your `*.ts.net` name, then fetches the cert. Caveats (verified):
- The MagicDNS name lands in **Certificate Transparency logs** (names only —
  access is still ACL-gated).
- Certs cover **only** your tailnet domain; bare hostnames aren't supported.
- 90-day expiry; `tailscale serve`/`funnel` provision + renew **automatically**,
  but standalone `tailscale cert` files are **your** job to renew.

## Serve and Funnel

`tailscale serve` exposes a local service to the **tailnet** over HTTPS (with an
automatic cert); `tailscale funnel` exposes it to the **public internet**.

```bash
tailscale serve 3000          # https://machine-name.tailnet-name.ts.net  → 127.0.0.1:3000 (tailnet only)
tailscale serve --bg 3000     # run in background
tailscale serve status
tailscale serve reset

tailscale funnel 3000         # same, but reachable from the open internet
```

- **Serve** keeps it private to the tailnet and forwards identity headers.
- **Funnel** is public, strips identity, supports **only ports 443 / 8443 /
  10000**, requires MagicDNS + HTTPS, and must be allowed in the policy file:

```json
{
  "nodeAttrs": [
    { "target": ["autogroup:member"], "attr": ["funnel"] }
  ]
}
```

## ACLs / zero-trust tailnet policy

The **tailnet policy file** (admin console → Access controls) is the heart of
[[Zero Trust Networking]] here: default-deny, then grant least-privilege paths
by **identity** and **tags** rather than IPs.

```json
{
  "tagOwners": {
    "tag:server": ["group:admins"],
    "tag:ci":     ["group:admins"]
  },
  "groups": {
    "group:admins": ["alice@example.com", "bob@example.com"]
  },
  "acls": [
    { "action": "accept", "src": ["group:admins"], "dst": ["tag:server:22,443"] },
    { "action": "accept", "src": ["tag:ci"],       "dst": ["tag:server:443"] }
  ],
  "ssh": [
    { "action": "check", "src": ["group:admins"], "dst": ["tag:server"], "users": ["root","autogroup:nonroot"] }
  ]
}
```

### Tags (machine identity)

Tags authenticate **non-user** devices (servers, CI runners) — like service
accounts. A tagged device drops user-based auth and key-expiry. Owners are
declared in `tagOwners`; apply at login:

```bash
sudo tailscale set --advertise-tags=tag:server          # or tag:server,tag:ci
sudo tailscale set --advertise-tags=                    # remove all tags
```

### Auth keys

Pre-auth keys register devices without an interactive browser login — for
containers, IoT, Terraform/IaC. Created in admin console → Keys (1–90 day
expiry, default 90). Types: **one-off**, **reusable** (treat like a secret),
**ephemeral** (auto-removes when offline), **pre-approved**, **tagged**.

```bash
sudo tailscale up --auth-key=tskey-auth-xxxxx --advertise-tags=tag:server
# ephemeral container node:
sudo tailscale up --auth-key=tskey-auth-xxxxx --advertise-tags=tag:ci   # + tailscaled --state=mem:
```

Keys are **case-sensitive**. A tagged node ignores auth-key expiry (key expiry
disabled by default for tags).

### OAuth clients

For automation that needs **long-lived** access, an OAuth client (client
ID + secret) mints fresh auth keys on demand — unlike a 90-day auth key it
doesn't expire. Created in admin console → Settings → OAuth clients with scoped
permissions (e.g. `auth_keys`, `devices:core`, `dns:read`).

- The `auth_keys` scope **requires** one or more tags at creation; those tags go
  on the keys it issues.
- Access tokens last **1 hour** (fixed). OAuth clients belong to the **tailnet**,
  not a user.
- CI pattern: the `get-authkey` helper reads `TS_API_CLIENT_ID` /
  `TS_API_CLIENT_SECRET` and emits an ephemeral, tagged key for the pipeline.

### Just-in-time (JIT) access

Tailscale offers several JIT paths: ACL `action: "check"` (re-auth via IdP per
[Tailscale SSH](#tailscale-ssh-how-it-works)), **device-posture** attributes
gating access, **SCIM** group sync so membership grants/revokes access, and
API-driven policy edits. The simplest, no-extra-cost JIT is `"check"` rules on
sensitive `ssh`/`acl` destinations.

## Tailnet lock

Tailnet lock makes trusted **signing nodes** cryptographically sign every new
node before peers will trust it — so a compromised control plane still can't
inject a rogue device. Requires ≥2 signing nodes (max 20), client ≥ v1.46, and a
**disablement secret** to turn off.

```bash
tailscale lock init        # bootstrap, designate signing keys
tailscale lock status
tailscale lock sign <node-key>
```

## Private AI / LLM access (Aperture)

Tailscale's AI story is **Aperture**, an AI gateway that fronts LLM providers
(OpenAI, Anthropic, Gemini, OpenAI-compatible, and **self-hosted** models) with
tailnet identity:

- Replaces scattered per-machine API keys with one **centralized**, audited
  gateway; the proxy injects the real provider key server-side.
- Routes by **model name** (`claude-sonnet-4-…`, `gpt-4o`, …) so tools only
  change their **base URL**.
- **Deny-by-default**: tailnet ACLs decide who can reach Aperture, then Aperture
  **grants** decide which models each user/group may call.
- Per-request telemetry (tokens, model, tool use) for audit/spend control.
- Self-hosted models can be proxied **without** exposing them publicly.

Beyond Aperture, the everyday pattern is simply putting a local LLM box (e.g.
[[Ollama]]) on the tailnet so only your devices reach it — no public exposure,
optionally TLS-fronted with `tailscale serve`.

## Firewalls — Fortinet cross-reference

> [!warning] Behind **Fortinet/FortiGate**, Tailscale nodes often fail to make
> direct connections and fall back to **DERP relays**. It tends to surface once
> **>5 users** sit behind the same firewall — Fortinet's NAT remaps the static
> WireGuard port unpredictably.

Fix in the tailnet policy file (admin console → Access controls) so clients use
a **random** WireGuard source port instead of the static **41641**:

```json
{
  // Tailnet policy file
  "randomizeClientPort": true
}
```

Also on FortiGate (see [[FortiGate Administration]]):
- **Exempt the Tailscale domains from SSL/deep inspection** — don't disable
  inspection wholesale. DPI re-signs the HTTPS control/coordination connection,
  which fails cert validation (Tailscale pins its endpoints). Add the domains to
  the SSL inspection profile's **Exempt from SSL Inspection → Addresses** list —
  best as a reusable **address group** so every profile/policy inherits it:

  ```
  tailscale.com              *.tailscale.com
  *.tailscale.io             controlplane.tailscale.com
  log.tailscale.com          derp.tailscale.com   *derp.tailscale.com
  ```

  Define each as an FQDN/wildcard-FQDN firewall address, bundle into one group
  (e.g. `TAILSCALE-EXEMPT`), then reference that group under *Exempt from SSL
  Inspection*. This keeps inspection on for everything else while letting the
  control plane and DERP relays through untouched.
- Don't block the **Tailscale application category** in app-control profiles.

## Where it fits a homelab / MSP

A mesh control plane lets management traffic (SSH, dashboards, APIs) ride the
encrypted overlay so it never crosses the public internet — a core
[[Zero Trust Networking]] pattern. For the self-hosted control plane (no
third-party coordination server), see [[Headscale]]; the data plane
(the WireGuard tunnels above) is identical either way.

## Related

- [[Headscale]] — run the control plane + DERP yourself.
- [[Zero Trust Networking]] · [[WireGuard]] · [[Mesh VPN]] — the model & tunnel.
- [[FortiGate Administration]] — the firewall side of the DERP fix.
- [[Linux Administration]] · [[Networking Reference]] — host & network setup.
