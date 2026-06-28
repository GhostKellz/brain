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

## 2026-06-28 additions (workstation, MSP, security)
- New [[Looking Glass]] reference — VFIO-passthrough Windows VM displayed in a
  lossless host window via **IVSHMEM** (zero-copy frames, no encode/no network
  hop). Grounded in the real Arch workstation (Ryzen 9950X3D, **RTX 5090 → guest**,
  Radeon iGPU stays on `amdgpu` for the host/LG client; clean IOMMU groups, no
  ACS patch). Covers IVSHMEM sizing, libvirt `<shmem>` XML, tmpfiles perms,
  host-app/client version coupling, input/audio/clipboard, and a dual-boot compare
  ("dual-booting is dead"). Wired into references `_index` (GPU & Wayland),
  [[DevOps and Homelab]] virtualization, [[Wiki Index]].
- New [[ScreenConnect]] reference (ConnectWise Control) — self-hosted remote
  support with **split exposure**: `screlay.<domain>` public (relay/session only,
  :8041), **web admin portal tailnet-only** (:8040 via [[Tailscale]]). Toolbox,
  clipboard passthrough, Backstage. Backlinks [[Azure Key Vault Code Signing]]
  (agent signing). New references `_index` "Remote access / RMM" group + [[Wiki Index]].
- New [[hermes-agent]] reference — Nous Research self-improving, model-agnostic
  autonomous agent (skills + bounded memory + FTS5 archive; point it at local
  [[Ollama]]/[[vLLM]]). New [[openshell]] concept — NVIDIA sandboxed,
  policy-governed agent runtimes (three-outcome egress, credential isolation,
  inference routing). Both wired into [[AI and Local LLMs]] agentic tooling +
  indexes.
- New [[ReFS]] reference — Windows CoW FS; **block cloning** powers Veeam Fast
  Clone, integrity streams + scrubber (off for VHDX), mirror-accelerated parity,
  **format repos ReFS-64K**. New [[Comet Backup]] reference — file-level + image/
  Hyper-V; client-side dedup + AES-256 to **your own Wasabi buckets** (not
  Comet-hosted), MSP portal/Storage Templates. Both tie into
  [[3-2-1 Backup Strategy]]; [[Veeam]] ↔ [[ReFS]] cross-linked.
- Also expanded [[Veeam]] (Fast Clone, Backup Copy Jobs, GFS, restore options,
  transport modes, 3-2-1-1-0, v13 deprecations).
- New [[Internal Domain Naming]] concept — RFC 6762 mDNS vs `.local`; why `.local`
  for AD/internal is deprecated (mDNS collisions, no public-CA certs, hybrid/SSO
  pain, TLD-collision risk) → use an owned public subdomain (`ad.example.com`).
  Backlinked from [[Active Directory Administration]].
- New [[Reverse Shells]] concept — bind vs reverse, mechanics (egress beats
  NAT/FW), PTY upgrade, threat-actor use, **authorized/internal** use cases, and
  blue-team defense (egress filtering, EDR, hunting). In [[Security]] domain.

## Last Updated
2026-06-28 — Workstation + MSP + security pass: [[Looking Glass]] (VFIO Windows VM
in a lossless host window — "dual-booting is dead"), [[ScreenConnect]] (split
exposure: public relay + tailnet-only admin), [[hermes-agent]] + [[openshell]]
(autonomous-agent harness + NVIDIA agent sandbox), [[ReFS]] + [[Comet Backup]]
(Veeam repo FS + file-level/Hyper-V backup to Wasabi), [[Internal Domain Naming]]
(why not `.local`) and [[Reverse Shells]]. Earlier today: Nix + backup stack
([[Nix]], [[Veeam]] expanded, [[Snapper]]).

## 2026-06-28 additions (Nix + backup stack)
- New [[Nix]] concept — Nix-the-package-manager/language/build-system + NixOS:
  the `/nix/store` content-addressed model, derivations/sandbox, flakes
  (inputs/outputs + lock), `nixos-rebuild` generations/rollback, channels-vs-flakes,
  home-manager/nix-darwin, the upstream/Determinate/Lix split, and an extensive
  **Nix vs Docker** comparison (build-time vs runtime reproducibility; Nix can
  *build* minimal OCI images). Wired into concepts `_index`, [[Wiki Index]],
  [[Linux and Systems]] packaging, [[DevOps and Homelab]] IaC; backlinked from [[ghostctl]].
- New [[Veeam]] reference (named "Veeam") — image-level VM backup: full/synthetic/
  incremental (Hyper-V **RCT**), **Scale-Out Backup Repository** (performance→
  capacity→archive tiers), offsite to **Azure/Wasabi** object storage,
  **immutability** (v13 entire-retention is performance-tier-only; capacity falls
  back to minimum-period), sealed-mode for mixing providers. Ties into
  [[3-2-1 Backup Strategy]]; backlinked from [[Hyper-V Administration]]. In
  references `_index` Backup + [[Wiki Index]].
- Promoted [[Snapper]] **entity → reference** and expanded it from the real
  `~/arch/btrfs/snapper` config: subvolume layout (@/@home/@snapshots/@pkg/@log),
  the `/etc/snapper/configs/root` retention knobs (NUMBER_LIMIT=10 + DAILY=7,
  min-age, space/free limits), **snap-pac** pacman hooks, timeline timers, manual
  live-ISO rollback, and the default-subvolume gotcha. Moved in entities `_index`
  → references `_index` Backup and re-slotted in [[Wiki Index]].

## 2026-06-28 additions (desktop, MSP stack, local LLM)
- Expanded [[KDE Plasma]] into a full page: layered theming (Aurorae), HDR/VRR on
  Plasma 6 for OLED, **Krohnkite** KWin tiling (layouts + real keybindings, float
  ignore-list, panic sequence), KWin shortcuts, Wayland-on-KDE. Wired in
  [[Linux and Systems]] desktop, [[Tiling Window Management]], [[Wayland]], [[NVIDIA]].
- New concepts [[Ghostty Terminal]] (GPU terminal in Zig; flat key=value config,
  Nerd Fonts) and [[Zsh and Powerlevel10k]] (Oh My Zsh + P10k instant prompt,
  `~/.zshrc.d` modular aliases, zoxide/fzf/direnv/vivid). Wired into
  [[Linux and Systems]] Tooling + concepts `_index` + [[Wiki Index]].
- New [[Huntress]] reference — managed EDR/ITDR/SIEM/SAT/ESPM/Managed Defender +
  24/7 SOC; absorbed the "Enable Microsoft Defender via PowerShell" runbook
  (workstation + Windows Server `MpCmdRun -WdEnable`/DISM) from
  [[Endpoint Security Tooling]] (now a pointer). In [[Security]] endpoint section.
- New [[Avanan]] reference — Check Point Harmony Email; API-based vs SEG,
  Monitor/Detect/Prevent ("virtual inline"), Click-Time Protection, SandBlast
  file scanning, Smart API for MSP (Client ID/Secret→JWT). In [[Security]] email
  + [[Cloud and Microsoft 365]].
- New [[Pax8]] reference — MSP cloud marketplace/distributor (billing engine,
  storefront, Microsoft CSP/NCE); "the Pax shell" is the **REST API** (OAuth
  client-credentials) — no official CLI; community PowerShell modules. In
  [[Cloud and Microsoft 365]] procurement.
- New [[vLLM]] concept — high-throughput serving engine: PagedAttention +
  continuous batching, OpenAI-compatible server, tensor/pipeline/data/expert
  parallelism, FP8/AWQ/GPTQ + FP8 KV cache, vLLM-vs-Ollama. Wired into
  [[Local LLM Inference]] (runtime-choice section), [[AI and Local LLMs]], concepts `_index`.
- Added a **Dependabot** section to [[CICD Workflows]] (version vs security
  updates, grouping, auto-merge, Renovate alternative).
- Wired the previously-orphaned [[Kubernetes]] concept into DevOps `_index`,
  [[DevOps and Homelab]] containers, and [[Wiki Index]]. Filled the entities
  index with 14 missing entity pages (ghost-kde,
  GhostCP, ghostport, GhostPSI, nvprime, Nitrogen, Nexus, Aventra, ctl365,
  Citadel, fgbackup, strix, zaur, zeppelin).

## 2026-06-28 additions (IaC, CI/CD, databases)
- New [[Ansible]] reference — agentless config management: inventory, playbooks,
  roles, ad-hoc, Jinja2/vars precedence, Ansible Vault, `ansible.cfg`; "Terraform
  builds, Ansible configures"; Tailscale-targeted inventory posture.
- New [[Terraform]] reference (note OpenTofu fork) — core loop (init/plan/apply),
  HCL, **state** + remote backends/locking, modules, workspaces; Proxmox provider
  + cloud-init + Cloudflare DNS-as-code; Terraform-vs-Ansible split.
- Expanded [[Proxmox]] cloud-init: added the actual `#cloud-config` format
  (users/packages/write_files/runcmd/power_state), the three data parts, and
  `cloud-init status/schema/clean` debugging (first-boot-only gotcha).
- New [[CICD Workflows]] concept — GitHub Actions (workflows/jobs/steps, matrix,
  **self-hosted runners** + the public-repo warning + homelab/Tailscale pattern)
  and GitLab CI (`.gitlab-ci.yml` stages, executors/runners); platform compare,
  pros/cons.
- New [[Databases]] concept — Postgres (the default), MariaDB/MySQL, SQLite,
  Redis/Valkey, TigerBeetle (ledgers, Zig), Turso/libSQL; SQL-vs-NoSQL, OLTP-vs-
  OLAP, per-engine pros/cons, a quick chooser, and architecture principles.

## 2026-06-28 additions (languages)
- New concept pages giving the general-purpose languages the same pros/cons
  treatment as Rust/Zig: [[Go]] (goroutines/channels, `go` toolchain, the
  `cobra`/`viper` + Charm `bubbletea`/`bubbles`/`lipgloss` stacks, web/db libs),
  [[Python]] (venvs on Linux + `pip`/`pipx`/`uv`/`poetry`, Flask/FastAPI,
  must-have libs, GIL trade-offs), [[JavaScript]] (language-vs-runtime: Node.js/
  Bun/Deno; npm/`node_modules`/lockfiles; npm vs pnpm vs yarn vs bun; TypeScript).
- New [[Cargo Workflow]] concept (resolves the long-dangling link) covering the
  full Rust toolchain: `rustup` (toolchains/targets), `rustc` (editions), `cargo`
  (build/test/clippy/fmt, features, profiles, workspaces, publish).
- Added depth to [[Rust]] (Go/Zig comparison, strengths/trade-offs, tooling
  pointer) and [[Zig]] (strengths/trade-offs). Wired all five into the
  [[Programming Languages]] hub, concepts `_index`, and [[Wiki Index]].

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
