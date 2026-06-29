---
type: domain
title: "DevOps and Homelab"
created: 2026-06-21
updated: 2026-06-22
tags:
  - domain
  - devops
  - homelab
status: developing
related:
  - "[[Linux and Systems]]"
  - "[[Networking]]"
  - "[[Cloud and Microsoft 365]]"
---

# DevOps and Homelab

Containers, virtualization, CI patterns, and the self-hosted homelab that ties
the brain together.

## Containers & orchestration
[[Docker]] · [[Kubernetes]] (k8s/k3s orchestration) · [[Portainer]] (archived) ·
[[NVIDIA Container Toolkit]] · [[NVIDIA Container Runtime Troubleshooting]] ·
[[Bolt]] (container runtime)

## Virtualization
[[Proxmox]] · [[Proxmox Backup Server]] (dedup VM/CT backup + S3 datastores) ·
[[Hyper-V Administration]] · [[VFIO GPU Passthrough]] ·
[[IOMMU and Device Groups]] · [[Looking Glass]] (passthrough Windows VM in a
lossless host window) · [[macOS Virtualization]]

## Self-hosting
[[Self-Hosted Services]] · [[NGINX Static Site Hosting]] · [[UniFi Controller]] ·
[[Let's Encrypt - Certbot|Let's Encrypt / Certbot]] ·
[[acme.sh - DNS-01 Certificates]]

## Observability
[[Uptime Kuma]] (uptime/synthetic) · [[Prometheus Monitoring]] · [[Grafana]] ·
[[Wazuh]] · [[CrowdSec]]

## ChatOps & alerting
[[Discord]] (webhooks/bots/alert hub) · [[SMTP2GO]] (outbound SMTP relay)

## CI/CD & infrastructure as code
[[CICD Workflows]] (GitHub Actions + self-hosted runners, GitLab CI) ·
[[Terraform]] (provisioning) · [[Ansible]] (configuration management) ·
[[Nix]] (declarative/reproducible builds, dev shells, NixOS fleet config)

## Data
[[Databases]] — choosing/architecting datastores (Postgres, MariaDB, SQLite,
Redis, TigerBeetle, Turso/libSQL).

## Toolkits
[[ghostctl]] — all-in-one sysadmin/DevOps/homelab toolkit.

## Build & packaging
[[Cargo Workflow]] · [[Pacman Hooks]] · [[PKGBUILD Templates]]
