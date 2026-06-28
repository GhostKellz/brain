---
type: reference
title: Fedora Administration
created: 2026-06-28
updated: 2026-06-28
tags:
  - linux
  - fedora
  - dnf
  - firewalld
  - selinux
  - sysadmin
status: developing
related:
  - "[[Linux Administration]]"
  - "[[RHEL, Rocky and Alma Administration]]"
  - "[[nftables Firewall]]"
---

Fedora administration: `dnf`, automatic updates, `firewalld`, and SELinux.
Cross-distro basics (SSH, swap, shell) live in [[Linux Administration]]. The
enterprise rebuilds share most of this — see [[RHEL, Rocky and Alma Administration]].

## Package Management (dnf)

```bash
sudo dnf upgrade --refresh           # sync metadata + upgrade everything
sudo dnf install <pkg>
sudo dnf remove <pkg>
sudo dnf autoremove                  # drop orphaned deps
sudo dnf search <term>
sudo dnf info <pkg>
dnf repoquery -l <pkg>               # files a package owns
sudo dnf install ./<file>.rpm        # local rpm + deps
dnf history                          # transaction log
sudo dnf history undo <id>           # roll back a transaction
```

### Repos & RPM Fusion

```bash
sudo dnf config-manager --add-repo <url>
# RPM Fusion (free + nonfree) — codecs, NVIDIA, etc.
sudo dnf install \
  https://download1.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm \
  https://download1.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm
```

## Automatic Updates (dnf-automatic)

Fedora's equivalent of unattended-upgrades:

```bash
sudo dnf install dnf-automatic
sudo vim /etc/dnf/automatic.conf     # set apply_updates = yes; emit_via = email/stdio
sudo systemctl enable --now dnf-automatic.timer
systemctl list-timers dnf-automatic.timer
```

```ini
# /etc/dnf/automatic.conf — key settings
[commands]
upgrade_type = security      # security | default
apply_updates = yes          # yes = install, no = download only
reboot = when-needed         # never | when-changed | when-needed
[emitters]
emit_via = email
```

> [!note] Keep update + reboot policy consistent with your other distros —
> compare the approach in [[Ubuntu Unattended Upgrades]].

## Firewall (firewalld)

Default frontend; drives nftables underneath (raw layer: [[nftables Firewall]]).

```bash
sudo systemctl enable --now firewalld
sudo firewall-cmd --state
sudo firewall-cmd --get-active-zones
sudo firewall-cmd --list-all                                   # current (runtime) zone
sudo firewall-cmd --add-service=ssh --permanent
sudo firewall-cmd --add-port=8080/tcp --permanent
sudo firewall-cmd --reload                                     # apply permanent rules
```

## SELinux

Fedora ships SELinux **enforcing** by default — don't disable it; fix labels.

```bash
getenforce                            # Enforcing | Permissive | Disabled
sudo setenforce 0                     # temporary permissive (debugging only)
sestatus
sudo ausearch -m avc -ts recent       # recent denials
sudo restorecon -Rv /path             # fix file contexts
sudo setsebool -P <boolean> on        # persistent boolean
# config: /etc/selinux/config (SELINUX=enforcing)
```

## Services & logs

```bash
sudo systemctl enable --now <svc>
sudo systemctl status <svc>
journalctl -u <svc> -e --no-pager
```

## Notes

- Binary-compatible enterprise rebuilds: [[RHEL, Rocky and Alma Administration]].
- Fedora Silverblue/IoT use `rpm-ostree` (image-based) instead of dnf — separate flow.
