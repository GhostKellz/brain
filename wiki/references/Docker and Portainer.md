---
type: reference
title: "Docker and Portainer"
created: 2026-06-21
updated: 2026-06-21
tags:
  - docker
  - portainer
  - containers
status: developing
related:
  - "[[Linux Administration]]"
  - "[[Let's Encrypt - Certbot|Let's Encrypt / Certbot]]"
  - "[[Nginx Reference]]"
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

## Notes

- Front Portainer/containers with TLS via [[Let's Encrypt - Certbot|Let's Encrypt / Certbot]] and [[Nginx Reference]].
