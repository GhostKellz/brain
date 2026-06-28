---
type: reference
title: Docker
aliases:
  - "Docker and Portainer"
  - "references/Docker-and-Portainer"
created: 2026-06-21
updated: 2026-06-28
tags:
  - docker
  - containers
  - compose
  - networking
  - dockerfile
  - sysadmin
status: developing
related:
  - "[[Linux Administration]]"
  - "[[Debian and Ubuntu Administration]]"
  - "[[nftables Firewall]]"
  - "[[Nginx Reference]]"
  - "[[Prometheus Monitoring]]"
  - "[[Self-Hosted Services]]"
  - "[[Portainer]]"
---

> [!key-insight] Generalized from field notes; host/client-specific values are placeholders. The container engine of record across the fleet — Compose stacks behind nginx, mounted onto host storage, on user-defined bridges.

Docker hub: install, image/container lifecycle, Dockerfiles & builds, Compose
stacks, volumes & bind mounts, networking (incl. DNS troubleshooting), the
Docker socket + socket-proxy, daemon tuning, and troubleshooting. Portainer is
split out (archival) → [[Portainer]].

## Install Docker Engine

Use Docker's official repo + the bundled Compose **plugin** (`docker compose`,
not the legacy `docker-compose` binary).

### Debian / Ubuntu

```bash
# Remove distro packages that conflict
for p in docker.io docker-doc docker-compose podman-docker containerd runc; do
  sudo apt remove -y "$p" 2>/dev/null; done

sudo apt update
sudo apt install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
  -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] \
https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" \
  | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io \
  docker-buildx-plugin docker-compose-plugin
```

> [!note] For Debian, swap `ubuntu` → `debian` in both the GPG and repo URLs.
> See [[Debian and Ubuntu Administration]].

### Arch

```bash
sudo pacman -S docker docker-buildx docker-compose
sudo systemctl enable --now docker.service
```

### Post-install

```bash
sudo usermod -aG docker "$USER"     # run docker without sudo (re-login required)
docker --version
docker compose version              # plugin form
docker run --rm hello-world         # smoke test
```

> [!warning] Adding a user to the `docker` group is **root-equivalent** — that
> user can mount the host FS into a container and escalate. Treat group
> membership as granting root.

## Image & container lifecycle

```bash
# Images
docker pull nginx:1.27
docker images                       # list local images
docker rmi <image>                  # remove image
docker image prune -a               # remove all unused images

# Containers
docker run -d --name web --restart unless-stopped -p 8080:80 nginx:1.27
docker ps                           # running
docker ps -a                        # incl. stopped
docker logs -f web                  # follow logs
docker exec -it web /bin/sh         # shell in (or bash)
docker stop web && docker rm web    # stop + remove
docker inspect web                  # full JSON (IP, mounts, env, …)
docker stats                        # live CPU/mem/net per container

# Wholesale cleanup (careful)
docker system df                    # disk usage by images/containers/volumes
docker system prune                 # dangling images, stopped containers, unused nets
docker system prune -a --volumes    # + unused images AND volumes (destructive)
```

> [!note] `--restart` policies: `no` (default), `on-failure[:N]`,
> `unless-stopped` (survives daemon restart but not an explicit stop),
> `always`. Use `unless-stopped` for most services.

## Dockerfiles & building

A `Dockerfile` is the recipe for an image. Order instructions **least- to
most-frequently-changing** so the layer cache stays warm (deps before source).

```dockerfile
# syntax=docker/dockerfile:1
# ---- build stage ----
FROM golang:1.23 AS build
WORKDIR /src
COPY go.mod go.sum ./
RUN go mod download            # cached unless go.mod/go.sum change
COPY . .
RUN CGO_ENABLED=0 go build -o /out/app ./cmd/app

# ---- runtime stage ----
FROM gcr.io/distroless/static:nonroot
COPY --from=build /out/app /app
USER nonroot:nonroot
EXPOSE 8080
ENTRYPOINT ["/app"]
```

Key instructions:

| Instruction | Purpose |
|-------------|---------|
| `FROM … AS name` | base image; name a stage for multi-stage builds |
| `WORKDIR` | set + create the working dir |
| `COPY` / `ADD` | copy files in (`ADD` also untars / fetches URLs — prefer `COPY`) |
| `RUN` | execute at build time → new layer |
| `ENV` / `ARG` | runtime env var / build-time var |
| `EXPOSE` | document a port (does **not** publish it) |
| `ENTRYPOINT` | fixed command; `CMD` supplies default args |
| `CMD` | default command/args (overridable at `docker run`) |
| `USER` | drop to non-root for the runtime |
| `HEALTHCHECK` | command Docker runs to mark container (un)healthy |

```bash
# Build (BuildKit is default in modern Docker)
docker build -t myapp:dev .
docker build -t myapp:dev --build-arg VERSION=1.2.3 .
docker build --no-cache -t myapp:dev .          # ignore layer cache

# Multi-platform via buildx
docker buildx build --platform linux/amd64,linux/arm64 -t myapp:1.2.3 --push .
```

Best practices:

- **Multi-stage builds** — compile in a fat image, copy only the artifact into a
  slim/distroless runtime. Smaller, fewer CVEs.
- **`.dockerignore`** — keep `.git`, `node_modules`, build output out of the
  build context (faster, smaller, avoids secret leaks).
- **Pin base tags** (`nginx:1.27`, not `latest`) for reproducibility.
- **Run as non-root** (`USER`) and prefer distroless/alpine where viable.
- Never bake secrets into layers — use `--secret` mounts or runtime env.

## Compose stacks

The default deployment unit. `compose.yaml` (or `docker-compose.yml`) in a
per-stack directory; manage with the `docker compose` plugin.

```yaml
# compose.yaml
services:
  app:
    image: myapp:1.2.3
    restart: unless-stopped
    env_file: .env
    ports:
      - "127.0.0.1:8080:8080"      # loopback-only; nginx fronts it
    volumes:
      - ./config:/etc/app:ro       # bind mount (host path)
      - app_data:/var/lib/app      # named volume
    depends_on:
      db:
        condition: service_healthy
    networks: [frontend, backend]

  db:
    image: postgres:16
    restart: unless-stopped
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/db_pw
    volumes:
      - db_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks: [backend]            # NOT on frontend → unreachable from edge
    secrets: [db_pw]

volumes:
  app_data:
  db_data:

networks:
  frontend:
  backend:
    internal: true                 # no outbound route

secrets:
  db_pw:
    file: ./secrets/db_pw.txt
```

```bash
docker compose up -d               # create/start in background
docker compose ps                  # status
docker compose logs -f app         # follow one service
docker compose pull && docker compose up -d   # update images + recreate
docker compose down                # stop + remove containers/networks
docker compose down -v             # … and named volumes (destructive)
docker compose exec app sh         # shell into a service
docker compose config              # render the merged, validated config
```

> [!key-insight] Compose auto-creates one network per project and registers each
> service under its name, so services resolve each other by **service name** with
> no extra config. Declare explicit networks only to **segment** tiers.

## Volumes & mounting folders

Two ways to persist/inject data:

| Type | Syntax | Use when |
|------|--------|----------|
| **Named volume** | `-v name:/path` | Docker-managed data you don't browse directly (DBs) |
| **Bind mount** | `-v /host/path:/path` | config files / content you edit on the host |
| **tmpfs** | `--tmpfs /path` | ephemeral in-RAM scratch (secrets, caches) |

```bash
docker volume create app_data
docker volume ls
docker volume inspect app_data     # shows Mountpoint under /var/lib/docker/volumes/
docker volume prune                # remove unused volumes

# Bind mount a host folder (read-only)
docker run -d --name web -v /srv/site:/usr/share/nginx/html:ro nginx

# Long form (explicit, recommended in scripts)
docker run -d --name web \
  --mount type=bind,src=/srv/site,dst=/usr/share/nginx/html,ro \
  nginx
```

> [!warning] Bind-mount permissions trip people up. The container's UID must be
> able to read/write the host path. Either align UIDs (`--user $(id -u):$(id -g)`),
> `chown` the host dir to the container's UID, or use a named volume. SELinux
> hosts ([[Fedora Administration]], [[RHEL, Rocky and Alma Administration]])
> need a `:z` (shared) or `:Z` (private) suffix on the mount or access is denied.

## Networking

The part everyone trips over. Docker gives each container a network stack; the
*driver* decides how that stack reaches other containers, the host, and the
outside world.

### Driver cheat-sheet

| Driver | Scope | Use when |
|--------|-------|----------|
| `bridge` | single host | default; isolated app stacks on one box |
| `host` | single host | container shares the host's network namespace (no isolation, no port mapping) |
| `macvlan` | single host | container needs a *real* IP on the LAN (its own MAC) |
| `ipvlan` | single host | like macvlan but shares the host MAC (switch port-security friendly) |
| `overlay` | multi-host (Swarm) | containers across hosts on one virtual L2 |
| `none` | single host | fully isolated, no networking |

### The default bridge vs a user-defined bridge

```bash
docker network ls                          # list networks
docker network inspect bridge              # the built-in default
```

> [!key-insight]
> Always create a **user-defined bridge** for an app stack — never rely on the
> default `bridge`. User-defined bridges give you **automatic DNS**: containers
> resolve each other by name. The default bridge does *not* (you'd be stuck with
> brittle `--link` or raw IPs).

```bash
# Create a user-defined bridge, attach containers, and they resolve by name
docker network create appnet
docker run -d --name db  --network appnet postgres:16
docker run -d --name web --network appnet myapp:latest
# inside `web`, the host `db` resolves to db's container IP automatically
```

### Port publishing (host ↔ container)

```bash
docker run -d -p 8080:80 nginx            # hostPort:containerPort
docker run -d -p 127.0.0.1:8080:80 nginx  # bind to loopback only — not 0.0.0.0
docker run -d -P nginx                     # publish all EXPOSEd ports to random highports
```

> [!warning]
> `-p 8080:80` binds `0.0.0.0` by default — reachable from the whole network, and
> Docker's iptables rules can **bypass a host ufw/firewalld policy**. Bind to
> `127.0.0.1:` and put a reverse proxy in front when a port shouldn't be public.
> See [[nftables Firewall]] for the `DOCKER-USER` chain to filter Docker traffic.

### macvlan — give a container its own LAN IP

```bash
docker network create -d macvlan \
  --subnet=192.0.2.0/24 --gateway=192.0.2.1 \
  -o parent=eth0 lan_macvlan
docker run -d --name dns --network lan_macvlan --ip 192.0.2.53 pihole/pihole
```

> [!key-insight]
> macvlan's gotcha: the **host cannot talk to its own macvlan containers** by
> default (the NIC won't hairpin its own MAC). If the host needs to reach them,
> add a `macvlan` shim interface on the host, or use `ipvlan l2` instead.

### Connecting, inspecting, cleaning up

```bash
docker network connect appnet web          # attach a running container to another net
docker network disconnect appnet web
docker network inspect appnet              # see subnet, gateway, attached containers
docker network prune                       # remove all unused networks
```

### Compose networks

```yaml
services:
  web:
    image: myapp:latest
    networks: [frontend, backend]
  db:
    image: postgres:16
    networks: [backend]          # db is NOT on frontend → unreachable from the edge

networks:
  frontend:
  backend:
    internal: true               # no outbound route to the host/internet
```

Compose auto-creates a default network per project, so services already resolve
each other by service name. Declare explicit networks only to **segment** tiers
(put the DB on a `backend` net the public-facing proxy can't see).

### DNS resolution troubleshooting

When containers can resolve each other but **not** the internet (or vice versa):

```bash
# 1. Does the container have a resolver + route at all?
docker exec web cat /etc/resolv.conf       # user-defined bridge → 127.0.0.11
docker exec web getent hosts db            # intra-stack name resolution
docker exec web nslookup example.com       # external DNS
docker exec web ping -c1 1.1.1.1           # raw L3 reachability (no DNS)
```

Docker's embedded DNS lives at `127.0.0.11` inside user-defined bridges and
forwards to the **host's** resolver. So if the host can't resolve, neither can
the container.

> [!key-insight] **No DNS / no outbound from containers? Check the host firewall
> first.** Docker inserts its rules into the `nat`/`filter` tables; a restrictive
> nftables/ufw/firewalld policy (or a custom default-drop `FORWARD` chain) can
> silently break container egress and DNS forwarding.

```bash
# Is forwarding on? Containers need it to reach anything off-host.
sysctl net.ipv4.ip_forward                 # must be 1

# Did the firewall flush Docker's chains? (common after a firewall reload)
sudo nft list chain ip nat DOCKER 2>/dev/null
sudo iptables -t nat -L DOCKER -n
sudo iptables -L FORWARD -n                 # look for a blanket DROP before Docker's rules

# ufw users: forwarding policy must allow it
sudo ufw default allow routed              # or scope it per-interface
# /etc/default/ufw → DEFAULT_FORWARD_POLICY="ACCEPT"

# Nuclear fix after a firewall reload clobbered Docker's rules:
sudo systemctl restart docker              # Docker re-installs its iptables/nft rules
```

See [[nftables Firewall]] — Docker traffic should be filtered via the
`DOCKER-USER` chain, which Docker leaves alone across restarts. Custom rules in
`DOCKER` get overwritten.

```bash
# Override container DNS explicitly if the host resolver is unreliable
docker run --dns 1.1.1.1 --dns 9.9.9.9 …
# or daemon-wide in /etc/docker/daemon.json: {"dns": ["1.1.1.1","9.9.9.9"]}
```

## The Docker socket & socket-proxy

`/var/run/docker.sock` is the daemon's control API. **Mounting it into a
container grants that container full root on the host** — anything that can talk
to the socket can spawn a privileged container and escape.

```yaml
# DANGEROUS — only for trusted management tools, and prefer the proxy below
volumes:
  - /var/run/docker.sock:/var/run/docker.sock:ro   # :ro is NOT a real safeguard
```

### docker-socket-proxy (least-privilege API)

Put a filtering proxy ([`tecnativa/docker-socket-proxy`](https://github.com/Tecnativa/docker-socket-proxy))
in front of the socket so management tools (Portainer, Traefik, Watchtower) only
get the endpoints they actually need — read-only by default.

```yaml
services:
  dockerproxy:
    image: tecnativa/docker-socket-proxy
    restart: unless-stopped
    environment:
      CONTAINERS: 1        # allow GET /containers
      IMAGES: 1
      NETWORKS: 1
      SERVICES: 0
      POST: 0              # deny all writes (no create/start/stop)
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
    networks: [dockerctl]
    # never publish a port — only reachable on the internal network

networks:
  dockerctl:
    internal: true
```

Point the consumer at `tcp://dockerproxy:2375` instead of the raw socket. Grant
`POST: 1` (and specific verbs) only to tools that genuinely need to mutate state.

## Daemon configuration (`/etc/docker/daemon.json`)

```json
{
  "log-driver": "json-file",
  "log-opts": { "max-size": "10m", "max-file": "3" },
  "default-address-pools": [
    { "base": "172.30.0.0/16", "size": 24 }
  ],
  "dns": ["1.1.1.1", "9.9.9.9"],
  "live-restore": true,
  "userland-proxy": false
}
```

- **`log-opts`** — cap container log growth (the #1 silent disk filler).
- **`default-address-pools`** — avoid bridge subnets colliding with the LAN/VPN.
- **`live-restore`** — containers keep running across daemon restarts/upgrades.

```bash
sudo systemctl restart docker   # required after editing daemon.json
```

## Troubleshooting quick hits

```bash
# Daemon health / why won't it start
sudo systemctl status docker
sudo journalctl -u docker -e            # see [[systemd]] for journald

# Container exits immediately
docker logs <container>                 # the actual error
docker inspect <container> --format '{{.State.ExitCode}} {{.State.Error}}'

# "no space left on device" — almost always image/log/volume bloat
docker system df
docker system prune -a --volumes        # destructive — know what you're deleting

# Permission denied on a bind mount → UID mismatch or SELinux (:z/:Z)
# Port already allocated → another container/process holds the host port
sudo ss -ltnp | grep :8080
```

## Notes

- Portainer (GUI) is split out and **archived** → [[Portainer]].
- Front containers with TLS via [[Nginx Reference]] +
  [[Let's Encrypt - Certbot|Let's Encrypt / Certbot]].
- Firewall interaction (the `DOCKER-USER` chain): [[nftables Firewall]].
- Container metrics (per-network IO included) via cadvisor → [[Prometheus Monitoring]].
- What runs in containers here: [[Self-Hosted Services]].
- Daemon under systemd (journald, restart, resource limits): [[systemd]].
