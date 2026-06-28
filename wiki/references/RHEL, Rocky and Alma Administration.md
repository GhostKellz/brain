---
type: reference
title: RHEL, Rocky and Alma Administration
created: 2026-06-28
updated: 2026-06-28
tags:
  - linux
  - rhel
  - rocky
  - alma
  - dnf
  - selinux
  - firewalld
  - sysadmin
status: developing
related:
  - "[[Linux Administration]]"
  - "[[Fedora Administration]]"
  - "[[nftables Firewall]]"
---

Enterprise family: Red Hat Enterprise Linux and its binary-compatible rebuilds
**Rocky Linux** and **AlmaLinux**. They share `dnf`, SELinux, and `firewalld`;
the main delta is subscription/repo management (see callout). Cross-distro basics
live in [[Linux Administration]]; the upstream/leading-edge cousin is
[[Fedora Administration]].

## Package Management (dnf)

```bash
sudo dnf upgrade --refresh
sudo dnf install <pkg>
sudo dnf remove <pkg>
sudo dnf autoremove
sudo dnf module list                 # AppStream module streams
sudo dnf module enable <name>:<stream>
dnf history                          # transaction log + rollback (history undo <id>)
```

## Subscription & repos

> [!note] RHEL vs Rocky/Alma — the real difference
> - **RHEL** requires a subscription: register with `subscription-manager`, then
>   enable repos. Free for dev via the Red Hat Developer program.
> - **Rocky / Alma** need no subscription — repos are preconfigured; skip
>   `subscription-manager` entirely. Otherwise commands are identical.

```bash
# RHEL only:
sudo subscription-manager register --username <user>
sudo subscription-manager attach --auto
sudo subscription-manager repos --enable=<repo-id>

# Common extra repo (all three): EPEL
sudo dnf install epel-release                 # Rocky/Alma
# RHEL: sudo subscription-manager repos --enable codeready-builder-for-rhel-9-x86_64-rpms
#       sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
```

## Automatic Updates (dnf-automatic)

```bash
sudo dnf install dnf-automatic
sudo vim /etc/dnf/automatic.conf     # upgrade_type, apply_updates, reboot
sudo systemctl enable --now dnf-automatic.timer
```

```ini
# /etc/dnf/automatic.conf — key settings
[commands]
upgrade_type = security
apply_updates = yes
reboot = when-needed
```

> [!note] Keep security-only vs all + reboot window consistent with your other
> distros — compare [[Ubuntu Unattended Upgrades]].

## Firewall (firewalld)

```bash
sudo systemctl enable --now firewalld
sudo firewall-cmd --add-service=ssh --permanent
sudo firewall-cmd --add-port=443/tcp --permanent
sudo firewall-cmd --reload
sudo firewall-cmd --list-all
```

Raw nftables layer beneath firewalld: [[nftables Firewall]].

## SELinux

Enforcing by default on the enterprise family — keep it on, fix contexts.

```bash
getenforce
sestatus
sudo ausearch -m avc -ts recent       # denials
sudo restorecon -Rv /path
sudo setsebool -P <boolean> on
sudo semanage port -a -t http_port_t -p tcp 8080   # allow nonstandard port (policycoreutils-python-utils)
# config: /etc/selinux/config
```

## Services & logs

```bash
sudo systemctl enable --now <svc>
journalctl -u <svc> -e --no-pager
```

## Notes

- Rocky and Alma are drop-in RHEL rebuilds; pick one per environment and stay
  consistent. Conversion tooling exists (`migrate2rocky`, `almalinux-deploy`).
