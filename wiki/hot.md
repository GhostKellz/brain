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
2026-06-28 — Uptime Kuma, SMTP2GO, Discord refs; Docker, memory-tuning & ARC enhanced.

## 2026-06-28 additions (memory & ARC)
- Expanded [[Linux Memory Tuning]] with a **gaming/performance profile**: disable
  *disk* swap → ZRAM-only + an OOM daemon (`systemd-oomd` PSI vs `earlyoom` RSS),
  low `swappiness`, `vm.max_map_count` (kernel 6.1+ defaults high; Fedora/SteamOS
  value for edge cases). Cross-linked from [[Linux Administration]]'s swap section.
  (zram/swap/oomd already lived in concepts: [[Linux Memory Tuning]] + [[ZRAM Swap]].)
- [[Proxmox]] ZFS ARC section already had the `zfs_arc_max` cap; added runtime
  inspect/tune (`arc_summary`, `arcstat`, `/proc/spl/kstat/zfs/arcstats`,
  `/sys/module/zfs/parameters/zfs_arc_max`).

## 2026-06-28 additions (monitoring & comms)
- New [[Uptime Kuma]] reference — the black-box/synthetic monitoring layer
  (complements Prometheus white-box). Catalog of monitor types (HTTP/keyword/JSON,
  TCP, ping, DNS, cert, Docker-via-socket-proxy, Push dead-man's-switch, DB, gRPC,
  MQTT, SNMP, real-browser), 90+ notification providers (chat/on-call/push/SMS/
  webhook/Apprise), status pages, maintenance windows, and **Alertmanager** both
  ways (Kuma→Alertmanager notifier; Prometheus scrape of Kuma `/metrics`). Plus
  the **remote-site/behind-NAT** pattern: an agent inside a client LAN pings local
  IPs and pushes outbound to a Push monitor (no inbound ports).
- New [[SMTP2GO]] reference (alias **SMTP**) — outbound SMTP relay/smarthost for
  app/alert mail. Create an SMTP user (auth is per-user, not account login), host
  `mail.smtp2go.com`, ports (587 STARTTLS / 465 SSL / 2525 fallback when 25/587
  blocked), msmtp + Uptime Kuma examples, SPF/DKIM deliverability → [[Email DNS Security]].
- New [[Discord]] reference — IT/ops chatops+alerting hub: webhooks (embeds, the
  `/github` & `/slack` URL trick, rate limits), bots (dev portal, intents,
  slash commands, discord.py example, least-privilege), alert producers
  ([[Uptime Kuma]]/[[Grafana]]/Alertmanager/CI), and the nginx **vanity invite
  redirect** (`discord.ghostkellz.sh` → `discord.gg/...`).
- Enhanced [[Docker]]: socket-proxy section now covers the **Tailscale bind via
  `.env`** (`SOCKET_PROXY_BIND_IP`) + **ufw allow on `tailscale0`** + GET-200/
  POST-403 verification + Uptime Kuma TCP/HTTP setup (no-trailing-space gotcha).
  New **container hardening** section (`read_only`/`tmpfs`/`cap_drop`/
  `no-new-privileges`/non-root), **volume & container backup** section (tar
  sidecar, DB dumps, hot-vs-cold, restic), and a **bridge-internals** subsection
  (veth/docker0/MASQUERADE/DNAT/ICC).

## 2026-06-28 additions (networking)
- New [[Linux Networking]] hub — the host-side L2 layer (complements
  [[Networking Reference]]'s L3 focus). Sections: runtime-vs-persistent renderers
  (ifupdown2/systemd-networkd/netplan/NetworkManager), iproute2 basics, bridges
  (runtime + persistent `/etc/network/interfaces` + systemd-networkd), bonds
  (active-backup/LACP/balance-alb), 802.1Q VLANs (sub-interfaces + VLAN-aware
  bridges), veth/namespaces, VXLAN (VNI/VTEP/underlay/overlay, MTU overhead),
  and a worked Proxmox example (vmbr0 mgmt + vmbr1-3 VLAN-aware on a 4-port NIC,
  `qm set --net0 ...tag=`, virtualized pfSense vmbr2=WAN/vmbr1=LAN).
- Expanded [[Proxmox]]: new `## Networking — bridges & passthrough` (vmbr overview
  pointing to [[Linux Networking]], `qm/pct set` tagging, `ifreload -a`, PCI/USB
  passthrough — WiFi adapter → Kali VM via IOMMU `hostpci` or USB `host=vendor:product`),
  `## Software-Defined Networking (SDN)` (zones Simple/VLAN/QinQ/VXLAN/EVPN, VNets,
  subnets, IPAM, `pvesh` apply), and `### Built-in DHCP (dnsmasq) + IPAM`
  (`dnsmasq@<zone>` units, PVE 8.1+). Cross-linked [[VFIO GPU Passthrough]].

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
