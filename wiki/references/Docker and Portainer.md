---
type: reference
title: "Docker and Portainer"
created: 2026-06-21
updated: 2026-06-22
tags:
  - docker
  - portainer
  - containers
  - networking
status: developing
related:
  - "[[Linux Administration]]"
  - "[[Let's Encrypt - Certbot|Let's Encrypt / Certbot]]"
  - "[[Nginx Reference]]"
  - "[[Prometheus Monitoring]]"
---

> [!key-insight] Generalized from field notes; host/client-specific values are placeholders.

Docker Engine install, common container ops, and Portainer (server + agent) deployment/upgrade.

## Install Docker Engine (Ubuntu)

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common

curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
  | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] \
https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" \
  | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io
docker --version
```

### Docker Compose (standalone binary)

```bash
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" \
  -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker-compose --version
```

## Common Operations

```bash
docker ps                                  # running containers
sudo docker volume create <name>           # create a named volume
docker exec -it <container> /bin/sh         # shell into a container
docker exec -it <container> bash
```

## Portainer Server

> [!warning] Never store a Portainer Business Edition license key in notes or a public wiki. Treat it as a secret (vault/password manager). Community Edition (`-ce`) needs no key.

```bash
# Create the data volume
sudo docker volume create portainer_data

# Pull (CE shown; swap -ce for -ee if licensed)
sudo docker pull portainer/portainer-ce:latest

# Run
sudo docker run -d -p 9443:9443 -p 8000:8000 \
  --name portainer --restart always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce:latest
```

### Upgrade Portainer

```bash
docker stop portainer
docker rm portainer
docker pull portainer/portainer-ce:latest
docker run -d -p 9443:9443 -p 8000:8000 \
  --name=portainer --restart=always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce:latest
```

## Portainer Agent (Edge/Remote Hosts)

```bash
docker run -d -p 9001:9001 \
  --name portainer_agent --restart=always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /var/lib/docker/volumes:/var/lib/docker/volumes \
  portainer/agent:latest
```

### Update the Agent in Place (one-liner)

```bash
sudo docker stop portainer_agent && sudo docker rm portainer_agent && \
sudo docker pull portainer/agent:latest && \
sudo docker run -d -p 9001:9001 --name portainer_agent --restart=always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /var/lib/docker/volumes:/var/lib/docker/volumes \
  portainer/agent:latest
```

## Docker networking

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

## Notes

- Front Portainer/containers with TLS via [[Let's Encrypt - Certbot|Let's Encrypt / Certbot]] and [[Nginx Reference]].
- Container metrics (per-network IO included) are exposed by cadvisor → see [[Prometheus Monitoring]].
