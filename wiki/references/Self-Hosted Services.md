---
type: reference
title: "Self-Hosted Services"
created: 2026-06-21
updated: 2026-06-28
tags:
  - selfhosted
  - pihole
  - kasm
  - dns
  - linux
status: developing
related:
  - "[[Let's Encrypt - Certbot|Let's Encrypt / Certbot]]"
  - "[[Docker]]"
  - "[[Tailscale]]"
  - "[[Proxmox]]"
  - "[[Uptime Kuma]]"
---

> [!key-insight] Generalized from field notes; all host/IP/topology values are
> placeholders. Specific home-lab inventory (which box runs what) stays in the
> private tier — this page is the public catalog of *how* to self-host each
> service, not a map of any one network.

A catalog of commonly self-hosted services and how to install/run them. Most run
in [[Docker]], reachable over [[Tailscale]] rather than public ports, with
[[Uptime Kuma]] watching them.

## Service map

| Category | Services | Deep-dive |
|----------|----------|-----------|
| DNS / network | Pi-hole, Technitium DNS | below |
| Mesh VPN / coordination | Headscale + DERP relays | below |
| VPN fallback | WireGuard site-to-site | below |
| Anonymity network | Tor relay | below |
| Edge / tunnels | Cloudflare Tunnel (cloudflared) | [[Cloudflare]] |
| Reverse proxy | Traefik (local Docker lab) | below |
| SSO / identity | Authentik, Keycloak | below |
| Remote desktop | Kasm Workspaces, RustDesk | below |
| Cameras / NVR | UniFi Protect (formerly Synology Surveillance) | below |
| Storage / NAS | Synology DSM | below |
| Object storage | MinIO (S3-compatible) | below |
| Media | Jellyfin (formerly Plex) | below |
| Git hosting | Gitea (+ self-hosted AUR), GitLab EE | below |
| Arch repo (planned) | Arch mirror + AUR package repo | below |
| Privacy frontends | Redlib | below |
| Time | NTP / chrony | below |
| CI / build | self-hosted GitHub Actions / GitLab runners | [[CICD Workflows]] |
| Local LLM | Ollama (CUDA), vLLM, Open WebUI, LiteLLM | [[Ollama Service Configuration]] · [[Local LLM Inference]] |
| Metrics / dashboards | Prometheus, Alertmanager, Grafana | [[Prometheus Monitoring]] · [[Grafana]] |
| Logs | Loki, syslog-ng | below |
| Uptime / synthetic | Uptime Kuma | [[Uptime Kuma]] |
| Security / SIEM | Wazuh | [[Wazuh]] |
| Web hosting | HestiaCP | [[HestiaCP]] |
| Home automation | Home Assistant | below |
| Knowledge / wiki | Wiki.js, Quartz (this brain) | below |
| Encrypted paste | PrivateBin | below |
| Secrets / passwords | Vaultwarden | below |
| Files / collab | Nextcloud | below |
| Notes sync | Obsidian LiveSync | below |
| Dev utilities | IT Tools | below |
| Containers | Docker (+ Portainer) | [[Docker]] · [[Portainer]] |

## DNS

### Pi-hole

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

> [!note] Pi-hole pairs well with Tailscale's `--accept-dns=false` so tailnet
> clients keep using your local resolver. See [[Tailscale]].

### Technitium DNS

A full authoritative + recursive DNS server with ad-blocking, DoH/DoT, and a web
UI — a more capable alternative to Pi-hole when you want to *run* DNS zones, not
just filter.

```bash
# Docker (recommended)
docker run -d --name technitium --restart unless-stopped \
  -p 53:53/udp -p 53:53/tcp -p 5380:5380/tcp \
  -v technitium-config:/etc/dns \
  technitium/dns-server

# or the native installer
curl -sSL https://download.technitium.com/dns/install.sh | sudo bash
```

Admin UI on `:5380`. Supports conditional forwarding, split-horizon, DNS-over-
HTTPS/TLS, and block lists; can be the recursive resolver behind a forwarder.
See [[Recursive DNS Resolution]].

## Mesh VPN coordination — Headscale

[[Headscale]] is the open-source re-implementation of Tailscale's coordination
server. Self-host it as a **lab** control plane; production stays on Tailscale's
hosted coordination (less to babysit, built-in DERP). See [[Tailscale]],
[[Mesh VPN]], [[WireGuard]].

```bash
docker run -d --name headscale --restart unless-stopped \
  -p 8080:8080 -p 9090:9090 \
  -v headscale-config:/etc/headscale \
  -v headscale-data:/var/lib/headscale \
  headscale/headscale:stable serve

# register a node (run the client with --login-server=<your-headscale-url>)
docker exec headscale headscale nodes register --user <user> --key <nodekey>
```

### Self-hosted DERP relays

When two nodes can't open a direct WireGuard path (hard NAT/firewall), traffic
falls back to a **DERP** relay. Tailscale runs a global DERP fleet; with
Headscale you point at your own via a custom DERP map:

```bash
# run a DERP server (TLS-terminating relay)
derper -hostname derp.<domain> -certmode letsencrypt -a :443
```

Reference it in `config.yaml` (`derp.paths` / `derp.urls`) so the tailnet hands
relayed flows to your relay instead of the public fleet — lower latency when your
relay is closer than the nearest public node.

> [!note] Newer alternative: Tailscale's **peer relays** let a node you nominate
> relay traffic over WireGuard (UDP) instead of DERP's TCP/HTTPS, cutting relay
> latency without standing up a full DERP server. Direct connections still win;
> relays are the fallback.

## WireGuard — site-to-site fallback

[[Tailscale]] is the primary VPN (business use, NAT traversal, ACLs). A plain
[[WireGuard]] tunnel is the **fallback**: a static site-to-site link between infra
and home that keeps working even if the Tailscale coordination plane is
unreachable. Fewer moving parts, no dependency on a control server.

```bash
sudo pacman -S wireguard-tools   # or apt install wireguard

# /etc/wireguard/wg0.conf (one side)
# [Interface]
# Address = <tunnel-ip>/24
# PrivateKey = <priv>
# ListenPort = 51820
# [Peer]
# PublicKey = <remote-pub>
# Endpoint = <remote-host>:51820
# AllowedIPs = <remote-subnet>/24      # routed networks for site-to-site
# PersistentKeepalive = 25
sudo systemctl enable --now wg-quick@wg0
```

> [!note] Set `AllowedIPs` to the **subnets** behind each peer (not just the
> tunnel IP) so the two sites route to each other. Open/forward UDP `51820` on the
> public side. Keep keys out of the public tier — config lives in [[Tailscale]]'s
> private homelab notes, not here. Concepts: [[Mesh VPN]], [[Zero Trust Networking]].

## Edge — Cloudflare Tunnel & DDNS

- **Cloudflare Tunnel (`cloudflared`)** — outbound-only daemon that publishes a
  local service through Cloudflare's edge with **no inbound ports** and the origin
  IP hidden. Full config + Zero Trust context in [[Cloudflare]].
- **Dynamic DNS** — when the ISP handed out a dynamic IP, a DDNS updater
  ([timothymiller/cloudflare-ddns](https://github.com/timothymiller/cloudflare-ddns))
  kept an A record pointed at the changing home IP via the Cloudflare API. **Now
  retired** — the ISP provides a **static IP**, so the record is fixed and the
  updater is no longer needed.

## Reverse proxy — Traefik

Container-native reverse proxy for the local Docker lab: it watches the Docker
socket and auto-routes to containers via labels (no hand-edited vhosts), with
built-in ACME/Let's Encrypt. Lighter-touch than [[Nginx Reference]] for a
dynamic, many-container setup.

```bash
docker run -d --name traefik --restart unless-stopped \
  -p 80:80 -p 443:443 -p 8080:8080 \
  -v /var/run/docker.sock:/var/run/docker.sock:ro \
  -v traefik-letsencrypt:/letsencrypt \
  traefik:v3 \
  --providers.docker --entrypoints.web.address=:80 \
  --entrypoints.websecure.address=:443
```

Then label app containers (`traefik.http.routers.<name>.rule=Host(...)`) to expose
them. nginx still fronts the static brain build; Traefik handles the dynamic lab.

## SSO / identity provider — Authentik & Keycloak

Self-hosted IdPs that put one login (OIDC/SAML/LDAP) in front of lab apps —
switched between the two depending on the need:

- **Authentik** — modern, friendlier UI, flexible flows/forward-auth; pairs well
  with Traefik/proxy forward-auth for apps without native SSO.
- **Keycloak** — heavyweight, enterprise-grade Red Hat project; the reference
  OIDC/SAML implementation when strict standards/federation matter.

```bash
# Authentik (compose recommended; needs Postgres + Redis)
docker run -d --name authentik-server --restart unless-stopped \
  -p 9000:9000 -p 9443:9443 \
  ghcr.io/goauthentik/server:latest server

# Keycloak (dev start; use a real DB + hostname config in prod)
docker run -d --name keycloak --restart unless-stopped \
  -p 8081:8080 \
  -e KEYCLOAK_ADMIN=<admin> -e KEYCLOAK_ADMIN_PASSWORD=<pass> \
  quay.io/keycloak/keycloak:latest start-dev
```

Back with Postgres → [[Databases]]; front with TLS ([[Nginx Reference]]/Traefik);
this is the identity layer for [[Zero Trust Networking]].

## Tor relay

Donate bandwidth to the Tor network by running a relay. A **middle/guard relay**
only passes encrypted traffic between other Tor nodes — it never connects to the
open internet on a user's behalf, so it carries no abuse/legal exposure (unlike an
**exit relay**, which does and should only be run with the right ISP/legal setup).

```bash
sudo pacman -S tor   # or apt install tor

# /etc/tor/torrc — minimal non-exit relay
# ORPort 9001
# Nickname <name>
# ContactInfo <email>
# RelayBandwidthRate 10 MB
# RelayBandwidthBurst 20 MB
# ExitRelay 0           # explicitly NOT an exit
sudo systemctl enable --now tor
```

> [!warning] Default to a non-exit relay (`ExitRelay 0`). Exit relays terminate
> traffic to the clear net and attract abuse complaints/legal attention — only run
> one on infrastructure provisioned for it. Give the relay its own host/IP, not a
> box tied to personal services.

Unrelated to [[Tailscale]]/[[Headscale]] (private mesh) — this is a public
anonymity-network contribution.

## Kasm Workspaces

Streamed containerized desktops/apps in the browser.

> [!note] The old multi-step `wget` flow is obsolete — current Kasm ships a
> single-server installer. Check the version on the
> [install docs](https://www.kasmweb.com/docs/develop/install/single_server_install.html).

```bash
cd /tmp
curl -O https://kasm-static-content.s3.amazonaws.com/kasm_release_1.17.0.7f020d.tar.gz
tar -xf kasm_release_1.17.0.7f020d.tar.gz
sudo bash kasm_release/install.sh
```

> [!note] On Arch/CachyOS this box uses ZRAM, not a swap file. If a target VM is
> RAM-tight, see the swap recipe in [[Linux Administration]] and
> [[Linux Memory Tuning]].

### Replace self-signed cert with Let's Encrypt

Issue a standalone cert (see [[Let's Encrypt - Certbot|Let's Encrypt / Certbot]]),
then swap Kasm's nginx cert/key and restart:

```bash
sudo /opt/kasm/bin/stop
cp /etc/letsencrypt/live/<fqdn>.<domain>/fullchain.pem /opt/kasm/current/certs/kasm_nginx.crt
cp /etc/letsencrypt/live/<fqdn>.<domain>/privkey.pem   /opt/kasm/current/certs/kasm_nginx.key
sudo /opt/kasm/bin/start
```

Set up a cron/`--deploy-hook` so renewals re-copy the cert and restart Kasm.

## Remote access — RustDesk

Self-hosted alternative to TeamViewer/AnyDesk: run your own relay + rendezvous
servers so remote-desktop sessions never touch a third-party cloud. Self-hosted at
one point for private remote support.

```bash
# rendezvous/ID server (hbbs) + relay server (hbbr)
docker run -d --name hbbs --restart unless-stopped --net host \
  -v rustdesk-data:/root rustdesk/rustdesk-server hbbs
docker run -d --name hbbr --restart unless-stopped --net host \
  -v rustdesk-data:/root rustdesk/rustdesk-server hbbr
```

Point clients at the server's public key + host; reach over [[Tailscale]] or open
the relay ports (`21115-21119`) deliberately.

## Storage — Synology NAS

Synology DSM is the appliance OS; the self-host-relevant pieces:

- **Shares** — SMB/NFS/AFP exports; the backup target for [[Restic Backup]] and
  the [[3-2-1 Backup Strategy]] (NAS = the on-site copy).
- **Container Manager** (Docker) — run containers directly on DSM 7.2+.
- **Snapshots** — Btrfs-based share snapshots + replication.
- **Access over [[Tailscale]]** — install the Tailscale package (or expose via a
  subnet router) rather than port-forwarding DSM to the internet.
- **Media server** — Jellyfin (free/open-source, no paywalled features) runs
  on the NAS via Container Manager; formerly Plex. Point it at the media shares:
  ```bash
  docker run -d --name jellyfin --restart unless-stopped \
    -p 8096:8096 \
    -v jellyfin-config:/config -v jellyfin-cache:/cache \
    -v /volume1/media:/media \
    jellyfin/jellyfin
  ```
  Reach it over [[Tailscale]]; UI on `:8096`.
- **Cameras** — formerly Synology **Surveillance Station**; since moved to
  **UniFi Protect** (see below).

## Cameras / NVR — UniFi Protect

Self-hosted video surveillance on UniFi hardware (Cloud Key / UNVR / Protect-
capable gateway): cameras record to local storage, no per-camera cloud
subscription, managed in the Protect app alongside the network stack. Replaced the
old Synology Surveillance Station setup. Network/controller side in
[[UniFi Controller]].

> [!note] Protect runs as an application on UniFi's own appliances rather than a
> generic Docker container — it's not a `docker run` service. Keep it on the
> management VLAN and reach it over [[Tailscale]] rather than exposing it.

## Object storage — MinIO

S3-compatible object storage you can run yourself — a drop-in target for backups,
artifacts, and apps expecting an S3 API (e.g. Terraform remote state, registry
blobs, Restic).

```bash
docker run -d --name minio --restart unless-stopped \
  -p 9000:9000 -p 9001:9001 \
  -e MINIO_ROOT_USER=<user> -e MINIO_ROOT_PASSWORD=<pass> \
  -v minio-data:/data \
  minio/minio server /data --console-address ":9001"
```

API on `:9000`, console on `:9001`. Back with [[Restic Backup]] / the
[[3-2-1 Backup Strategy]]; front with [[Nginx Reference]] + TLS.

> [!note] Alternatives worth weighing: **Garage** (Rust, lightweight, multi-site
> geo-replication), **SeaweedFS** (huge-file/object + volume model), and
> **RustFS** (newer Rust S3 server). MinIO is the mature default; Garage/RustFS
> trade ecosystem maturity for a smaller, simpler footprint.

## Git hosting

- **Gitea** — lightweight self-hosted Git forge (issues, PRs, packages, Actions).
  Runs locally for private repos and mirrors:
  ```bash
  docker run -d --name gitea --restart unless-stopped \
    -p 3000:3000 -p 222:22 \
    -v gitea-data:/data \
    gitea/gitea:latest
  ```
  - **Self-hosted AUR** — host a personal Arch package repo as a Gitea repo:
    publish `PKGBUILD`s and/or a pacman repo db so boxes can `pacman -S` your
    builds. See [[PKGBUILD Templates]], [[Pacman Hooks]], [[Arch Linux Administration]].
- **GitLab Enterprise Edition** — full DevOps platform (CI/CD, registry, pages)
  self-hosted on a cloud server for heavier pipelines. Pipeline detail in
  [[CICD Workflows]].

> [!note] Planned: a dedicated **Arch mirror + AUR package repo** — rsync a
> pacman mirror locally (fast LAN updates, less public-mirror load) and serve a
> signed custom repo of personal/AUR builds. See [[PKGBUILD Templates]],
> [[Pacman Hooks]], [[Arch Linux Administration]].

## Privacy frontend — Redlib

Redlib is a private, JS-free front-end for Reddit (no ads/tracking, no account).

```bash
docker run -d --name redlib --restart unless-stopped \
  -p 8080:8080 \
  quay.io/redlib/redlib:latest
```

Put it behind [[Nginx Reference]] + TLS; reach over [[Tailscale]] for personal use.

## Time — NTP

Run a local time source so LAN hosts (and air-gapped/Tailscale-only boxes) stay in
sync without hammering public pools. `chrony` is the modern daemon:

```bash
sudo pacman -S chrony   # or apt install chrony
sudo systemctl enable --now chronyd
chronyc sources -v      # check upstream + reachability
```

Point clients at the server (`server <host> iburst` in `chrony.conf`); accurate
time is a prerequisite for TLS, Kerberos/AD, and log correlation. See
[[Linux Administration]].

## Self-hosted CI runners

Run your own GitHub Actions / GitLab runners for GPU jobs and private-network
access (full pipeline detail in [[CICD Workflows]]).

- **As a systemd service on a Linux VM** — register the runner, then install it
  as a unit so it survives reboots:
  ```bash
  ./config.sh --url https://github.com/<org>/<repo> --token <reg-token>
  sudo ./svc.sh install && sudo ./svc.sh start   # GitHub runner as a service
  ```
- **GPU CI via VFIO passthrough** — pass a GPU into the runner VM with
  [[VFIO GPU Passthrough]] / [[IOMMU and Device Groups]], then use the
  [[NVIDIA Container Toolkit]] so jobs get the GPU inside ephemeral containers
  (`--gpus all`). Lets quick containerized builds/tests use real hardware.

> [!warning] Self-hosted runners belong on **private** repos only — a fork PR can
> run arbitrary code on the runner. Prefer ephemeral runners. See the security
> note in [[CICD Workflows]].

## Local LLM serving

- **Ollama (CUDA)** — easy local model serving/embeddings; service tuning in
  [[Ollama Service Configuration]], model choice in [[Local LLM Inference]].
- **vLLM** — high-throughput OpenAI-compatible inference server for larger
  models / concurrent requests:
  ```bash
  docker run --gpus all -p 8000:8000 \
    vllm/vllm-openai:latest --model <hf-model-id>
  ```
  Exposes an OpenAI-style `/v1` API; pair with the [[NVIDIA Container Toolkit]].
- **Open WebUI** — self-hosted chat UI over Ollama / any OpenAI-compatible
  endpoint (a ChatGPT-style front-end for local + remote models):
  ```bash
  docker run -d --name open-webui --restart unless-stopped \
    -p 3000:8080 \
    -e OLLAMA_BASE_URL=http://<ollama-host>:11434 \
    -v open-webui:/app/backend/data \
    ghcr.io/open-webui/open-webui:main
  ```
- **LiteLLM** — proxy that puts one OpenAI-compatible API in front of many
  providers (Ollama/vLLM/OpenAI/Anthropic/OpenRouter), with routing, keys, and
  spend tracking. Handy as the single endpoint Open WebUI and apps point at.

## Observability & logging

The white-box monitoring stack (black-box/synthetic is [[Uptime Kuma]]):

- **[[Prometheus Monitoring]]** — metrics scraping + alert rules.
- **Alertmanager** — routes/dedupes Prometheus alerts to Discord/email/etc. (both
  Prometheus and [[Uptime Kuma]] can feed it).
- **[[Grafana]]** — dashboards over Prometheus, Loki, and other datasources.
- **Loki** — log aggregation queried in Grafana; pairs with Promtail/agents.
- **syslog-ng** — central syslog collector; ship host/appliance logs to Loki or
  files.

## Security / SIEM

- **[[Wazuh]]** — self-hosted SIEM/XDR: agent-based file-integrity, log analysis,
  and detections, with its own dashboard.

## Web hosting

- **[[HestiaCP]]** — open-source hosting control panel (nginx/Apache, PHP, mail,
  DNS) for self-hosted web hosting.

## Home automation — Home Assistant

Local-first home automation hub (HA OS, supervised, container, or core). Run it in
[[Docker]] for a quick start:

```bash
docker run -d --name homeassistant --restart unless-stopped \
  --network host \
  -v ha-config:/config \
  -e TZ=<timezone> \
  ghcr.io/home-assistant/home-assistant:stable
```

UI on `:8123`. Keeps automations/state local (no cloud dependency); reach it over
[[Tailscale]] and put it behind a reverse proxy ([[Nginx Reference]]) for TLS.

## Knowledge / wikis

- **Quartz (this brain)** — the public tier of this Obsidian vault is self-hosted
  as a [Quartz](https://quartz.jzhao.xyz) static site (build host → static HTML →
  nginx). It's the canonical example here: markdown in git, rendered to a fast
  read-only site. See the deploy overview in the repo README.
- **Wiki.js** — a self-hosted database-backed wiki (editor, auth, search) for
  content that needs in-browser editing/permissions rather than a git/markdown
  flow:
  ```bash
  docker run -d --name wikijs --restart unless-stopped \
    -p 3000:3000 \
    -e DB_TYPE=postgres -e DB_HOST=<db> -e DB_USER=<u> \
    -e DB_PASS=<p> -e DB_NAME=wiki \
    requarks/wiki:2
  ```
  Backed by Postgres → [[Databases]]; front it with [[Nginx Reference]] + TLS.
- **Obsidian LiveSync** — self-hosted live sync for Obsidian vaults (the editing
  layer behind this brain) using a CouchDB backend, so devices sync in real time
  without Obsidian Sync's hosted service:
  ```bash
  docker run -d --name obsidian-couchdb --restart unless-stopped \
    -p 5984:5984 \
    -e COUCHDB_USER=<u> -e COUCHDB_PASSWORD=<p> \
    -v couchdb-data:/opt/couchdb/data \
    couchdb:latest
  ```
  The Self-hosted LiveSync community plugin points at this CouchDB; front with
  [[Nginx Reference]] + TLS and reach over [[Tailscale]]. This is the **private
  editing/sync** path — distinct from the public Quartz build above.

## Dev utilities — IT Tools

Self-hosted collection of everyday developer/sysadmin utilities (encoders,
hash/JWT/UUID generators, converters, formatters, network helpers) in one offline
web app — handy to keep these off random third-party sites.

```bash
docker run -d --name it-tools --restart unless-stopped \
  -p 8080:80 \
  ghcr.io/corentinth/it-tools:latest
```

Static front-end, no backend/state; put it behind [[Nginx Reference]] + TLS and
reach over [[Tailscale]] for internal use.

## Encrypted paste — PrivateBin

Zero-knowledge pastebin: the server only stores ciphertext — encryption/decryption
happen in the browser with the key in the URL fragment (never sent to the server),
so it's safe for sharing secrets/encrypted notes.

```bash
docker run -d --name privatebin --restart unless-stopped \
  -p 8080:8080 \
  -v privatebin-data:/srv/data \
  privatebin/nginx-fpm-alpine
```

Set expiry/burn-after-reading per paste; put it behind [[Nginx Reference]] + TLS
and reach it over [[Tailscale]] for internal-only use.

## Secrets / passwords — Vaultwarden

Lightweight Rust re-implementation of the Bitwarden server — a self-hosted vault
for lab passwords, API keys, and TOTP that's compatible with the official
Bitwarden clients/extensions but far lighter to run.

```bash
docker run -d --name vaultwarden --restart unless-stopped \
  -p 8080:80 \
  -v vaultwarden-data:/data \
  vaultwarden/server:latest
```

> [!warning] The vault is only as safe as its perimeter. Put it behind
> [[Nginx Reference]] + TLS (Bitwarden clients require HTTPS), reach it over
> [[Tailscale]] rather than the public internet, disable open signups after
> setup (`SIGNUPS_ALLOWED=false`), and back up `/data` → [[Restic Backup]] /
> [[3-2-1 Backup Strategy]]. The blobs are encrypted, but losing them = losing
> every credential.

## Files / collaboration — Nextcloud

Self-hosted file sync + share (Drive/Dropbox alternative) with calendar, contacts,
and an app ecosystem. Pairs the NAS shares with cross-device sync clients:

```bash
docker run -d --name nextcloud --restart unless-stopped \
  -p 8080:80 \
  -v nextcloud-data:/var/www/html \
  nextcloud:stable
```

Back it with Postgres/MariaDB ([[Databases]]) + Redis for caching; front with
[[Nginx Reference]] + TLS, reach over [[Tailscale]], back up the data dir →
[[Restic Backup]].

## Notes

- Container-based deployments: [[Docker]]. Run DBs on named volumes and back them
  up → [[Databases]], [[Restic Backup]].
- **Container management:** mostly plain [[Docker]]/Compose, with a [[Portainer]]
  instance available for a web UI over stacks/containers when handy.
- Watch everything with [[Uptime Kuma]]; alert through Alertmanager → [[Discord]].
