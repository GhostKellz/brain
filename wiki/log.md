---
type: meta
title: "Operations Log"
created: 2026-06-21
updated: 2026-06-22
tags:
  - meta
  - log
status: stable
related:
  - "[[Wiki Index]]"
  - "[[Hot Cache]]"
---

# Operations Log

Append-only. Newest entries at the TOP. Each entry: `## [YYYY-MM-DD] operation | title`.

## [2026-06-22] expand | Reference-authoring pass
- Sources mined (sanitized): `~/arch` (nvim config, restic/btrfs), `~/fortigate`
  (FortiOS CLI cheatsheet), `~/proxmox` (cloud-init/PBS/ZFS), `~/heimdall-stack`
  (Prometheus/Grafana/Wazuh/CrowdSec/Loki observability compose).
- New references (7): [[acme.sh - DNS-01 Certificates]],
  [[Azure Key Vault Code Signing]], [[Neovim Cheatsheet]],
  [[Prometheus Monitoring]], [[Grafana]], [[Wazuh]], [[CrowdSec]].
- Expanded (5): [[Proxmox Administration]] (cloud-init, LXC, VM, ZFS, PBS),
  [[FortiGate Administration]] (interfaces→VPN→SD-WAN→HA→flow-debug),
  [[Restic Backup]] (excludes, check, restore), [[Docker and Portainer]]
  (Docker networking: bridge/host/macvlan/overlay, DNS, port publishing),
  [[Btrfs Subvolume Layout]] (ssd/discard=async/space_cache=v2 mount opts).
- Wiring: added [[acme.sh - DNS-01 Certificates]], the observability quartet,
  and the new cheatsheets to [[Wiki Index]], references `_index`, and the
  DevOps/Security/Linux/Cloud/Web domain hubs.
- Tiering: every real host/IP/client/domain/credential placeholdered. Generic
  vendor/tool names (FortiGate, Prometheus, Wazuh) kept; leak guard clean.

## [2026-06-21] seed | Initial brain seed
- Sources: `~/arch` (public-safe subset), `/data/projects` (85 repos),
  the three web properties (`ckelley.dev`, `ghostkellz.sh`, `cktechx.com`),
  and the sanitized CK Cheatsheet.
- Pages created: 49 entities, 38 concepts, 39 references, 3 sources, 8 domain
  hubs, plus [[Brain Overview]], [[Wiki Index]], [[Hot Cache]], this log, and
  the meta dashboard. ~140 pages total.
- Backbone: [[Brain Overview]], [[Wiki Index]], [[Hot Cache]], 8 domain hubs,
  anchor entities [[GhostKellz]] and [[CK Technology LLC]].
- Collision resolved: split Tailscale into [[Tailscale]] (entity) +
  [[Tailscale Operations]] (runbook).
- Tiering: PUBLIC vault. `heimdall-stack`/`security` specifics held back to the
  private tier. Cheatsheet sanitized — client domains/IPs/keys removed or
  placeholdered. Client docs stay in Hudu.
- Key insight: one operator across three layers (MSP, homelab, OSS) unified by
  *local-first infra, Rust for apps, [[Zig]] for engines*.
