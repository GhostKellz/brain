---
type: reference
title: Portainer
created: 2026-06-21
updated: 2026-06-28
tags:
  - portainer
  - docker
  - containers
  - archive
status: archived
related:
  - "[[Docker]]"
  - "[[Nginx Reference]]"
---

> [!note] **Archived.** Portainer is no longer actively used here — kept for
> reference. Day-to-day container work is CLI + Compose; see [[Docker]]. If you
> deploy Portainer, front it with the docker-socket-proxy pattern from
> [[Docker]] rather than mounting the raw socket.

Portainer is a web GUI for managing Docker hosts, Compose stacks, volumes, and
networks. Server runs on the management host; Agents run on remote hosts the
server controls.

## Portainer Server

> [!warning] Never store a Portainer Business Edition license key in notes or a
> public wiki. Treat it as a secret (vault/password manager). Community Edition
> (`-ce`) needs no key.

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

## Notes

- Container engine reference (install, Compose, networking, socket-proxy): [[Docker]].
- Front the Portainer UI with TLS via [[Nginx Reference]].
