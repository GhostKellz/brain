---
type: reference
title: Debian and Ubuntu Administration
created: 2026-06-28
updated: 2026-06-28
tags:
  - linux
  - debian
  - ubuntu
  - apt
  - ufw
  - sysadmin
status: developing
related:
  - "[[Linux Administration]]"
  - "[[Ubuntu Unattended Upgrades]]"
  - "[[nftables Firewall]]"
  - "[[Proxmox Administration]]"
---

> [!key-insight] One note for the whole apt family. Debian and Ubuntu share `apt`/`dpkg`/systemd; the deltas (snap, netplan defaults, ESM/Pro, release cadence) are small — see the differences callout at the bottom.

Debian/Ubuntu-family server administration: packages, firewall, networking, and
update policy. Cross-distro basics (SSH, swap, shell) live in [[Linux Administration]].

## Package Management

```bash
sudo apt update && sudo apt full-upgrade -y     # full upgrade (handles removals)
sudo apt upgrade -y                             # conservative (no removals)
sudo apt install <pkg>
sudo apt remove <pkg>                           # keep config
sudo apt purge <pkg>                            # remove config too
sudo apt autoremove --purge                     # drop orphaned deps + old kernels
sudo apt-get install -f                         # fix broken/half-installed deps

apt search <term>
apt show <pkg>
apt list --installed
dpkg -l | grep <pkg>                            # query installed (dpkg level)
dpkg -L <pkg>                                   # files a package owns
sudo dpkg -i <file>.deb && sudo apt-get install -f   # install local .deb + deps
```

### Held / pinned packages

```bash
sudo apt-mark hold <pkg>        # freeze at current version
sudo apt-mark unhold <pkg>
apt-mark showhold
```

## Unattended Upgrades

Baseline patching posture on every server — full runbook:
**[[Ubuntu Unattended Upgrades]]**.

```bash
sudo apt install unattended-upgrades apt-listchanges
sudo dpkg-reconfigure --priority=low unattended-upgrades
```

## Uncomplicated Firewall (UFW)

UFW is the default frontend; it drives nftables underneath (see [[nftables Firewall]]
for the raw layer / advanced rules).

```bash
sudo apt install ufw
sudo ufw status
sudo ufw status numbered
sudo ufw allow ssh                # or: sudo ufw allow <port>
sudo ufw allow 'Nginx Full'       # named app profiles
sudo ufw allow 'OpenSSH'
sudo ufw enable
sudo ufw delete <rule-number>
```

> [!warning] Run `sudo ufw allow <ssh-port>` before `ufw enable` on a remote box.

## Networking (netplan — Ubuntu)

Ubuntu Server uses **netplan** (`/etc/netplan/*.yaml`); Debian uses
`/etc/network/interfaces` or systemd-networkd.

```yaml
# /etc/netplan/01-netcfg.yaml  (Ubuntu)  — YAML, 2-space indent
network:
  version: 2
  ethernets:
    eth0:
      addresses: [192.0.2.10/24]
      routes:
        - to: default
          via: 192.0.2.1
      nameservers:
        addresses: [1.1.1.1, 9.9.9.9]
```

```bash
sudo netplan try        # apply with auto-rollback on timeout (safe)
sudo netplan apply
```

## Services & logs

```bash
sudo systemctl enable --now <svc>
sudo systemctl status <svc>
journalctl -u <svc> -e --no-pager
journalctl -b -p err --no-pager        # this-boot errors
```

## Debian vs Ubuntu — the deltas

> [!note] What actually differs between the two
> - **snap**: preinstalled on Ubuntu (`snap install/list/refresh`); not on Debian.
>   Remove/avoid on minimal servers if undesired.
> - **netplan**: Ubuntu default; Debian uses `ifupdown`/`systemd-networkd`.
> - **ESM / Ubuntu Pro**: Ubuntu's extended security maintenance — enable with
>   `sudo pro attach <token>`; relevant to `unattended-upgrades` ESM origins.
>   Debian has no equivalent (use stable + `-security` suite).
> - **Release cadence**: Ubuntu LTS (5-yr) / interim (9-mo) vs Debian stable
>   (~2-yr) + oldstable. Pick `Allowed-Origins` accordingly.
> - **Codename macros**: both expose `${distro_id}`/`${distro_codename}` to APT
>   config, so unattended-upgrades stanzas are portable across the two.

## Notes

- Proxmox nodes are Debian-based — apt workflow applies, but pin/blacklist the
  PVE kernel. See [[Proxmox Administration]].
