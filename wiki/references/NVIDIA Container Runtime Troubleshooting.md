---
type: reference
title: "NVIDIA Container Runtime Troubleshooting"
created: 2026-06-21
updated: 2026-06-21
tags:
  - nvidia
  - docker
  - containers
  - troubleshooting
  - arch-linux
status: seed
related:
  - "[[NVIDIA Container Toolkit]]"
  - "[[Pacman Hooks]]"
---

# NVIDIA Container Runtime Troubleshooting

Configuring and fixing the NVIDIA container stack on Arch + Docker/containerd.
The [[NVIDIA Container Toolkit]] injects driver libs into containers via a CDI
spec; that spec goes stale on driver updates, which is the source of most breakage.

## Key config files

| File | Role |
|------|------|
| `/etc/docker/daemon.json` | Registers the nvidia runtime with Docker |
| `/etc/containerd/conf.d/99-nvidia.toml` | Registers it with containerd |
| `/etc/cdi/nvidia.yaml` | CDI spec mapping current driver libs/devices |

## Register the runtime

```bash
sudo nvidia-ctk runtime configure --runtime=docker
sudo nvidia-ctk runtime configure --runtime=containerd
sudo systemctl restart docker containerd
```

## Regenerate the CDI spec (after any driver change)

```bash
sudo nvidia-ctk cdi generate --output=/etc/cdi/nvidia.yaml
```

## Verify

```bash
docker run --rm --gpus all nvidia/cuda:12.6.3-base-ubuntu24.04 nvidia-smi
```

## Breakage 1 — driver version mismatch (most common)

Symptom after a driver upgrade:

```
OCI runtime create failed: ... open /usr/lib/libEGL_nvidia.so.<OLD_VERSION>:
no such file or directory
```

Cause: `/etc/cdi/nvidia.yaml` still points at the old driver's `.so` paths.

```bash
sudo nvidia-ctk cdi generate --output=/etc/cdi/nvidia.yaml
sudo systemctl restart docker
docker compose up -d   # recreate affected containers
```

## Breakage 2 — TTRPC shim error after containerd upgrade

```
failed to start shim: ... failed to create TTRPC connection: unsupported protocol
```

Cause: stale nvidia shim registration after a containerd version bump.

```bash
sudo nvidia-ctk runtime configure --runtime=containerd
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart containerd docker
docker rm -f <container>   # if left in a broken state
```

## Full recovery (nuclear option)

```bash
docker rm -f <stuck-container>
sudo nvidia-ctk runtime configure --runtime=containerd
sudo nvidia-ctk runtime configure --runtime=docker
sudo nvidia-ctk cdi generate --output=/etc/cdi/nvidia.yaml
sudo systemctl restart containerd docker
docker run --rm --gpus all nvidia/cuda:12.6.3-base-ubuntu24.04 nvidia-smi
```

## Prevention — automate CDI regen with a pacman hook

> [!key-insight]
> A [[Pacman Hooks|pacman hook]] that regenerates the CDI spec whenever the
> driver package upgrades turns the most common breakage into a non-event.

```ini
# /etc/pacman.d/hooks/nvidia-container-toolkit.hook
[Trigger]
Operation = Install
Operation = Upgrade
Type = Package
Target = nvidia-utils
Target = nvidia-open-dkms

[Action]
Description = Regenerating NVIDIA CDI spec for container toolkit...
When = PostTransaction
Exec = /usr/bin/nvidia-ctk cdi generate --output=/etc/cdi/nvidia.yaml
NeedsTargets
```

## Diagnostics

```bash
nvidia-smi | head -5                    # current driver version
pacman -Q nvidia-container-toolkit containerd docker
grep "driver" /etc/cdi/nvidia.yaml | head -3   # what CDI thinks the driver is
docker run --rm --runtime=runc hello-world     # test runc alone (bypass nvidia)
```

## Related

- [[NVIDIA Container Toolkit]] — how the injection works
- [[Pacman Hooks]] — the automation mechanism
