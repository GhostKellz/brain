---
type: meta
title: "Hot Cache"
aliases:
  - "Hot Cache"
created: 2026-06-21
updated: 2026-06-28
tags:
  - meta
status: developing
related:
  - "[[Wiki Index]]"
  - "[[Brain Overview]]"
---

# Recent Context

## Last Updated
2026-06-28 — Linux Server Hardening bible added; Btrfs layout expanded.

## 2026-06-28 additions (hardening)
- New [[Linux Server Hardening]] — the authoritative, example-heavy hardening
  guide (the "bible" applied to every production Linux server). Sections: full
  `sshd` policy, SSH/admin behind [[Tailscale]] (the house posture), deny-by-
  default firewall, Fail2Ban (only if public), reverse-proxy/TLS, sysctl
  hardening, patching, package/service minimization, logging/auditd/[[Wazuh]],
  file integrity, least-exposure, example prod layouts, review checklist. Links
  out to the deep-dive notes rather than duplicating. Cross-linked from
  [[Linux Administration]] (baseline → full posture).
- Expanded [[Btrfs]] layout: subvolume basics (create/list/snapshot/set-default),
  the `.snapshots` directory structure (snapper `N/snapshot` + `info.xml`; a
  snapshot *is* a subvolume), and a distro-standard layouts table (Arch flat,
  openSUSE nested, Ubuntu/Timeshift, Fedora).

## 2026-06-28 additions (filesystems)
- Consolidated the three Btrfs notes ([[Btrfs Subvolume Layout]],
  [[Btrfs Restore From Snapshot]], [[Btrfs Troubleshooting]]) into one [[Btrfs]]
  reference (old titles + old slugs kept as aliases/redirects). Sections:
  subvolume layout, restore from snapshot, troubleshooting. Concept/entity notes
  [[Btrfs Snapshots]] and [[Snapper]] stay separate.
- New full [[ZFS]] reference (pools/vdevs, dataset properties, snapshots/clones,
  send/receive replication, scrub/disk-replace, ARC tuning, sanoid/syncoid).
  Generic ZFS moved out of Proxmox; [[Proxmox]] keeps a pointer + the
  Proxmox-specific ARC cap.
- Renamed [[Proxmox Administration]] → [[Proxmox]] (alias + old-slug redirect
  kept). Repointed all active backlinks.

## 2026-06-28 additions (Docker)
- Renamed [[Docker and Portainer]] → [[Docker]] (alias + old-slug redirect kept)
  and rebuilt it as a comprehensive container hub: modern install (`docker
  compose` plugin), image/container lifecycle, Dockerfiles & multi-stage builds,
  Compose stacks, volumes/bind mounts (incl. SELinux `:z/:Z`), networking
  (preserved driver/bridge/macvlan content), DNS-resolution troubleshooting
  (check `ip_forward` + nftables/ufw `DOCKER-USER` chain), the docker socket +
  socket-proxy least-privilege pattern, `daemon.json` tuning, troubleshooting.
- Split Portainer into its own note [[Portainer]], marked **archived** (no longer
  actively used; kept for reference). Server/Agent install + upgrade moved there.
- Repointed all active [[Docker and Portainer]] backlinks → [[Docker]]; added
  [[Portainer]] to index/_index/DevOps hub catalogs.

## 2026-06-28 additions
- Restructured Linux admin around a hub: [[Linux Administration]] is now an
  overarching index (cross-distro basics: SSH, swap, shell, system info) that
  links out to per-distro runbooks.
- New per-distro notes: [[Debian and Ubuntu Administration]],
  [[Fedora Administration]], [[RHEL, Rocky and Alma Administration]],
  [[openSUSE Administration]]. Arch kept separate as the daily driver
  ([[Arch Linux Administration]], now with a firewall cross-link).
- Debian+Ubuntu kept in ONE note (deltas in a callout); Ubuntu vs Debian not
  split. Firewall content stays DRY in [[nftables Firewall]]; each distro note
  links to it and names its default frontend (ufw / firewalld / nftables).
- Fedora/RHEL/openSUSE notes carry baseline content + `[!todo]` flags where house
  defaults (update/reboot policy, baseline package set) still need filling.
- Renamed [[systemd Drop-in Overrides]] → [[systemd]] (alias + old-slug redirect
  kept) and expanded it into a full leveraging-systemd reference: unit model,
  systemctl/journald, drop-ins, targets, cgroup resource control, service
  hardening (`systemd-analyze security`), user services. Repointed all backlinks.
- Earlier: [[Ubuntu Unattended Upgrades]] runbook (security-only default,
  auto-reboot 03:30, mail on-change).

## Prior (2026-06-22)
2026-06-22 — Reference-authoring pass: 7 new runbooks + 4 expansions, all
sanitized from `~/arch`, `~/fortigate`, `~/proxmox`, and `~/heimdall-stack`.

## 2026-06-22 additions
- New references: [[acme.sh - DNS-01 Certificates]] (Cloudflare + Azure DNS),
  [[Azure Key Vault Code Signing]], [[Neovim Cheatsheet]],
  [[Prometheus Monitoring]], [[Grafana]], [[Wazuh]], [[CrowdSec]].
- Expanded: [[Proxmox Administration]] (cloud-init/LXC/VM/ZFS/PBS),
  [[FortiGate Administration]] (interfaces→routing→policy→VPN→SD-WAN→HA→debug),
  [[Restic Backup]] (excludes/verify/restore), [[Docker and Portainer]]
  (Docker networking deep-dive), [[Btrfs Subvolume Layout]] (SSD mount opts).
- All host/IP/client/domain identifiers placeholdered; leak guard clean.

## Key Recent Facts
- The brain (`~/brain` → `github.com/ghostkellz/brain`, public) is now seeded with
  ~130 cross-linked pages across 8 domains.
- Unifying thesis captured: local-first infra, **Rust for apps / [[Zig]] for engines**.
- Sources mined: `~/arch` (public-safe subset), `/data/projects` (85 repos),
  the three web properties, and the (sanitized) CK Cheatsheet.

## Recent Changes
- Created backbone: [[Brain Overview]], [[Wiki Index]], 8 domain hubs,
  [[GhostKellz]], [[CK Technology LLC]].
- Seeded entities (47), concepts (38), references (39), sources (3).
- Resolved a filename collision: split Tailscale into [[Tailscale]] (entity) +
  [[Tailscale]] (runbook).

## Tiering reminders
- PUBLIC vault. `heimdall-stack`/`security` specifics excluded → private tier.
- Cheatsheet was sanitized: client domains/IPs/keys removed or placeholdered.
- Client docs stay in Hudu, never here.

## Active Threads
- Done: seed committed; MCPVault filesystem MCP wired for both Claude Code and
  Codex; local hybrid retrieval (BM25 + ollama `nomic-embed-text` rerank) built.
- Tiering update: one vault only — public content is tracked/pushed; private notes
  live in gitignored `private/` paths, local-only (no separate GitLab vault).
- Pending later phases: MemPalace eval, Quartz publish to `brain.ckelley.dev`,
  optional Graphify code-graph visuals. Eventually capture config in
  `~/arch/dotfiles/claude`.
