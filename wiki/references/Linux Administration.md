---
type: reference
title: "Linux Administration"
created: 2026-06-21
updated: 2026-06-21
tags:
  - linux
  - ssh
  - firewall
  - sysadmin
status: developing
related:
  - "[[Arch Linux Administration]]"
  - "[[Docker and Portainer]]"
  - "[[Let's Encrypt - Certbot|Let's Encrypt / Certbot]]"
  - "[[Networking Reference]]"
---

> [!key-insight] Generalized from field notes; host/client-specific values are placeholders.

General Debian/Ubuntu-family Linux administration: SSH, firewall, packages, and system info.

## System Info

```bash
lsb_release -d        # distro & version
uname -mrs            # kernel / arch
uptime -p             # human-readable uptime
ip a                  # network interfaces (legacy: ifconfig)
echo $PATH            # show PATH
man -k <keyword>      # search man pages (apropos)
```

## Package Management & Updates

```bash
sudo apt update && sudo apt full-upgrade -y
sudo apt-get install -f          # fix broken/half-installed deps

# Unattended security upgrades
sudo apt install unattended-upgrades
sudo dpkg-reconfigure --priority=low unattended-upgrades
```

## SSH

### Install client/server

```bash
sudo apt update && sudo apt upgrade
sudo apt install openssh-client openssh-server
sudo systemctl status ssh
sudo ufw allow ssh
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

- Change the listen `Port` to a non-default value (under 1024 if you want it privileged).
- Enable `PubkeyAuthentication` and disable password login once keys work.
- `PermitRootLogin` — leave `no` unless you have a deliberate reason; prefer sudo.

```bash
sudo vim /etc/ssh/sshd_config     # edit Port / auth settings
sudo systemctl restart sshd
```

## Uncomplicated Firewall (UFW)

```bash
sudo apt install ufw
sudo ufw status
sudo ufw status numbered
sudo ufw allow <port>
sudo ufw allow 'Apache Full'      # named app profiles
sudo ufw allow 'Nginx Full'
sudo ufw enable
sudo ufw delete <rule-number>
```

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

```bash
sudo dd if=/dev/zero bs=1M count=1024 of=/mnt/1GiB.swap
sudo chmod 600 /mnt/1GiB.swap
sudo mkswap /mnt/1GiB.swap
sudo swapon /mnt/1GiB.swap
cat /proc/swaps                                    # verify
echo '/mnt/1GiB.swap swap swap defaults 0 0' | sudo tee -a /etc/fstab   # persist
```

## Notes

- Arch/pacman-specific workflows live in [[Arch Linux Administration]].
- Containers: [[Docker and Portainer]]. TLS: [[Let's Encrypt - Certbot|Let's Encrypt / Certbot]].
