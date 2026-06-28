---
type: reference
title: openSUSE Administration
created: 2026-06-28
updated: 2026-06-28
tags:
  - linux
  - opensuse
  - suse
  - zypper
  - yast
  - firewalld
  - sysadmin
status: developing
related:
  - "[[Linux Administration]]"
  - "[[nftables Firewall]]"
  - "[[Btrfs Snapshots]]"
---

openSUSE (Leap / Tumbleweed) administration: `zypper`, YaST, `firewalld`, and
snapper-backed Btrfs rollback. Cross-distro basics live in [[Linux Administration]].

## Package Management (zypper)

```bash
sudo zypper refresh                  # refresh repos
sudo zypper update                   # apply patches/updates (Leap)
sudo zypper dup                      # distribution upgrade — REQUIRED on Tumbleweed (rolling)
sudo zypper install <pkg>
sudo zypper remove <pkg>
sudo zypper search <term>
sudo zypper info <pkg>
zypper se --installed-only           # list installed
sudo zypper install ./<file>.rpm     # local rpm + deps
```

> [!key-insight] Leap uses `zypper update`; **Tumbleweed (rolling) uses
> `zypper dup`** — using plain `update` on Tumbleweed can break dependency state.

### Repos

```bash
zypper lr -d                         # list repos (detailed)
sudo zypper addrepo <url> <alias>
sudo zypper modifyrepo --refresh <alias>
```

## YaST

SUSE's unified config tool (TUI or GUI) — network, services, users, partitions,
firewall, bootloader:

```bash
sudo yast                            # ncurses menu
sudo yast2 <module>                  # e.g. yast2 lan, yast2 firewall
```

## Automatic Updates

```bash
# Leap: zypper-based automatic patching
sudo zypper install yast2-online-update-configuration   # or roll a systemd timer around 'zypper patch'
```

> [!note] Pick a mechanism (timer around `zypper patch`/`dup` vs YaST) and keep
> update + reboot policy consistent with your other distros — compare
> [[Ubuntu Unattended Upgrades]]. MicroOS uses `transactional-update` (atomic,
> reboot-to-apply) instead.

## Firewall (firewalld)

Modern openSUSE defaults to firewalld (older releases used SuSEfirewall2).

```bash
sudo systemctl enable --now firewalld
sudo firewall-cmd --add-service=ssh --permanent
sudo firewall-cmd --add-port=443/tcp --permanent
sudo firewall-cmd --reload
```

Raw nftables layer beneath: [[nftables Firewall]].

## Snapshots & rollback

openSUSE ships **snapper** + Btrfs out of the box with a layout tuned for full
system rollback (including bootloader integration):

```bash
sudo snapper list
sudo snapper rollback <number>        # roll the system back to a snapshot
```

See [[Btrfs Snapshots]] for the underlying mechanics.

## Services & logs

```bash
sudo systemctl enable --now <svc>
journalctl -u <svc> -e --no-pager
```

## Notes

- **Leap** = stable, RHEL-cadence-ish; **Tumbleweed** = rolling (use `dup`);
  **MicroOS** = immutable/transactional.
