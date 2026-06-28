---
type: reference
title: Linux Administration
created: 2026-06-21
updated: 2026-06-28
tags:
  - linux
  - ssh
  - sysadmin
  - index
status: developing
related:
  - "[[Arch Linux Administration]]"
  - "[[Debian and Ubuntu Administration]]"
  - "[[Fedora Administration]]"
  - "[[RHEL, Rocky and Alma Administration]]"
  - "[[openSUSE Administration]]"
  - "[[nftables Firewall]]"
  - "[[Linux Server Hardening]]"
---

> [!key-insight] Overarching Linux hub. Truly cross-distro basics live here; package management, firewall frontends, and update policy live in the per-distro runbooks linked below.

General-purpose Linux administration that applies regardless of distro — SSH,
shell, swap, system info — plus the index to distro-specific admin notes.

## By distro family

| Family | Runbook | Packages | Default firewall |
|--------|---------|----------|------------------|
| Arch (daily driver) | [[Arch Linux Administration]] | `pacman` / AUR | nftables direct |
| Debian / Ubuntu | [[Debian and Ubuntu Administration]] | `apt` / `dpkg` | `ufw` |
| Fedora | [[Fedora Administration]] | `dnf` | `firewalld` |
| RHEL / Rocky / Alma | [[RHEL, Rocky and Alma Administration]] | `dnf` | `firewalld` |
| openSUSE | [[openSUSE Administration]] | `zypper` | `firewalld` |

Firewall internals (the raw ruleset every frontend ultimately drives) are in
[[nftables Firewall]]. Each distro note links back to it.

## System Info

```bash
hostnamectl                 # host, kernel, OS, hardware vendor (systemd)
cat /etc/os-release         # distro & version (portable across distros)
lsb_release -d              # distro & version (where lsb-release is installed)
uname -mrs                  # kernel / arch
uptime -p                   # human-readable uptime
ip a                        # network interfaces (legacy: ifconfig)
echo $PATH                  # show PATH
man -k <keyword>            # search man pages (apropos)
```

## SSH

### Install client/server

The package name is the same family-to-family; only the installer differs (see
each distro note for the exact command):

```bash
# Debian/Ubuntu:  sudo apt install openssh-server
# Fedora/RHEL:    sudo dnf install openssh-server
# Arch:           sudo pacman -S openssh
# openSUSE:       sudo zypper install openssh

sudo systemctl enable --now sshd       # 'ssh' on Debian/Ubuntu, 'sshd' elsewhere
sudo systemctl status sshd
```

### Key-based authentication

```bash
ssh-keygen -t ed25519                 # preferred
ssh-keygen -t rsa                     # legacy/compat
mkdir ~/.ssh && chmod 700 ~/.ssh

# Copy public key to a remote host
ssh-copy-id <user>@<host>

# From Windows PowerShell
scp $env:USERPROFILE\.ssh\id_ed25519.pub <user>@<host>:~/.ssh/authorized_keys
```

> [!note] Store key passphrases in a password manager — never in your SSH client app.

### Hardening (`/etc/ssh/sshd_config`)

Universal across distros:

- Change the listen `Port` to a non-default value (under 1024 if you want it privileged).
- Enable `PubkeyAuthentication` and disable password login once keys work.
- `PermitRootLogin` — leave `no` unless you have a deliberate reason; prefer sudo.

```bash
sudo vim /etc/ssh/sshd_config     # edit Port / auth settings
sudo systemctl restart sshd
```

> [!warning] Open the new SSH port in the firewall *before* restarting, or you'll
> lock yourself out. Firewall frontend is distro-specific — see the table above.

> [!key-insight] This is the *baseline*. For the full production posture (complete
> `sshd` policy, deny-by-default firewall, SSH behind [[Tailscale]], TLS, sysctl
> hardening, monitoring, review checklist) follow **[[Linux Server Hardening]]**.

## Shell Snippets

```bash
# Append a line to a file only if not already present (idempotent)
grep -qxF "<line>" <file> || echo "<line>" >> <file>

# Make a script executable and run it
chmod +x script.sh
./script.sh

# Output redirection
cat > output.txt        # overwrite
cat >> output.txt       # append
```

## Add Swap (file-backed)

Cross-distro (systemd/fstab based):

```bash
sudo dd if=/dev/zero bs=1M count=1024 of=/mnt/1GiB.swap
sudo chmod 600 /mnt/1GiB.swap
sudo mkswap /mnt/1GiB.swap
sudo swapon /mnt/1GiB.swap
cat /proc/swaps                                    # verify
echo '/mnt/1GiB.swap swap swap defaults 0 0' | sudo tee -a /etc/fstab   # persist
```

> [!note] On Arch/CachyOS this box runs zram swap instead — see [[ZRAM Swap]].

## Notes

- systemd is the common layer across all of these — see [[systemd Timers]] and
  [[systemd]].
- Containers: [[Docker]]. TLS: [[Let's Encrypt - Certbot|Let's Encrypt / Certbot]].
