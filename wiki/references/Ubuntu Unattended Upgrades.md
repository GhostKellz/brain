---
type: reference
title: Ubuntu Unattended Upgrades
created: 2026-06-28
updated: 2026-06-28
tags:
  - linux
  - ubuntu
  - debian
  - apt
  - sysadmin
  - patching
status: developing
related:
  - "[[Debian and Ubuntu Administration]]"
  - "[[Linux Administration]]"
  - "[[systemd Timers]]"
  - "[[Proxmox Administration]]"
---

> [!key-insight] Standard server-hardening step applied to **all** Debian/Ubuntu servers. Values below are house defaults; placeholders mark site-specific entries.

Automatic, unattended security (and optionally regular) package updates via the
`unattended-upgrades` package, driven by APT's periodic systemd timers. This is
the baseline patching posture on every Ubuntu/Debian server.

## Install

```bash
sudo apt update
sudo apt install unattended-upgrades apt-listchanges
sudo dpkg-reconfigure unattended-upgrades   # enable; writes 20auto-upgrades
```

- `unattended-upgrades` — the upgrade engine + systemd timers.
- `apt-listchanges` — surfaces changelog/NEWS entries (mailed or logged) so you
  know what actually changed.
- `dpkg-reconfigure` (answer **Yes**) writes a minimal
  `/etc/apt/apt.conf.d/20auto-upgrades`. Use `--priority=low` to be prompted for
  more options.

## Config files

Two files do the work; both live in `/etc/apt/apt.conf.d/`.

| File | Purpose |
|------|---------|
| `20auto-upgrades` | Enables the periodic timers (the *when*). |
| `50unattended-upgrades` | Defines *what* gets upgraded and the behaviour. |

### `20auto-upgrades` — enable the timers

```conf
APT::Periodic::Update-Package-Lists "1";      // apt update daily
APT::Periodic::Unattended-Upgrade "1";        // run unattended-upgrade daily
APT::Periodic::Download-Upgradeable-Packages "1";
APT::Periodic::AutocleanInterval "7";         // clean apt cache weekly
```

### `50unattended-upgrades` — behaviour & policy

Key stanzas we set on servers (`sudo vim /etc/apt/apt.conf.d/50unattended-upgrades`):

```conf
Unattended-Upgrade::Allowed-Origins {
    "${distro_id}:${distro_codename}";
    "${distro_id}:${distro_codename}-security";
    "${distro_id}ESMApps:${distro_codename}-apps-security";
    "${distro_id}ESM:${distro_codename}-infra-security";
//  "${distro_id}:${distro_codename}-updates";   // uncomment for ALL updates, not just security
};

// Never auto-touch these (kernel pinning, DB engines, etc.)
Unattended-Upgrade::Package-Blacklist {
//  "linux-";
//  "postgresql-";
};

// Housekeeping
Unattended-Upgrade::Remove-Unused-Kernel-Packages "true";
Unattended-Upgrade::Remove-New-Unused-Dependencies "true";
Unattended-Upgrade::Remove-Unused-Dependencies "true";

// Reboots
Unattended-Upgrade::Automatic-Reboot "true";
Unattended-Upgrade::Automatic-Reboot-WithUsers "true";
Unattended-Upgrade::Automatic-Reboot-Time "03:30";

// Notifications (requires a working MTA / msmtp)
Unattended-Upgrade::Mail "alerts@<domain>";
Unattended-Upgrade::MailReport "on-change";   // always | only-on-error | on-change

// Resilience
Unattended-Upgrade::AutoFixInterruptedDpkg "true";
Unattended-Upgrade::MinimalSteps "true";
Unattended-Upgrade::InstallOnShutdown "false";
```

> [!note] `Allowed-Origins` vs `Origins-Pattern`: Ubuntu ships `Allowed-Origins`,
> Debian ships `Origins-Pattern`. Match whatever the stock file uses on that distro.

### House defaults (what we set on every server)

- **Security only** by default — leave `-updates` commented unless the role wants
  full updates.
- **Auto-reboot ON** at `03:30` (stagger times across a fleet to avoid a
  simultaneous reboot storm).
- **Blacklist `linux-`** on boxes where we control kernel upgrades manually
  (e.g. DKMS/out-of-tree modules, [[Proxmox Administration]] nodes).
- **Mail on-change** to the alerts mailbox so patching is auditable.
- **Autoremove unused deps + old kernels** to keep `/boot` from filling.

## Verify & test

```bash
# Dry run — shows what WOULD be upgraded, changes nothing
sudo unattended-upgrade --dry-run --debug

# Confirm the periodic config APT will actually use
apt-config dump APT::Periodic

# Timers that drive it
systemctl list-timers 'apt-daily*'
systemctl status apt-daily.timer apt-daily-upgrade.timer

# Force a real run now
sudo unattended-upgrade -v
```

## Logs

```bash
# Upgrade history & actions
sudo less /var/log/unattended-upgrades/unattended-upgrades.log
sudo less /var/log/unattended-upgrades/unattended-upgrades-dpkg.log

# Did a pending reboot get flagged?
ls -l /var/run/reboot-required*
cat /var/run/reboot-required.pkgs 2>/dev/null
```

## Timer mechanics

`unattended-upgrades` is triggered by two systemd timers (not cron):

- `apt-daily.timer` → `apt-daily.service` — refreshes package lists & downloads.
- `apt-daily-upgrade.timer` → `apt-daily-upgrade.service` — applies upgrades.

Both have randomized delays. To shift the window, drop in an override rather than
editing the unit (see [[systemd]] drop-in overrides):

```bash
sudo systemctl edit apt-daily-upgrade.timer
```

```ini
[Timer]
OnCalendar=
OnCalendar=*-*-* 03:00
RandomizedDelaySec=30m
```

## Notes

- General Debian/Ubuntu admin (SSH, UFW, packages): [[Linux Administration]].
- Arch uses pacman + hooks instead — see [[Arch Linux Administration]] and
  [[Pacman Hooks]].
- Stagger `Automatic-Reboot-Time` and timer `OnCalendar` across the fleet so
  servers don't all reboot at once.
