---
type: reference
title: Linux Server Hardening
created: 2026-06-28
updated: 2026-06-28
tags:
  - linux
  - hardening
  - security
  - ssh
  - firewall
  - sysadmin
  - server
status: developing
related:
  - "[[Linux Administration]]"
  - "[[Debian and Ubuntu Administration]]"
  - "[[nftables Firewall]]"
  - "[[Tailscale]]"
  - "[[Nginx Reference]]"
  - "[[Ubuntu Unattended Upgrades]]"
  - "[[Sysctl Performance Tuning]]"
  - "[[Wazuh]]"
  - "[[CrowdSec]]"
---

> [!key-insight] The hardening bible. This is the standard checklist + worked examples applied to **every production Linux server**. Cross-distro basics (install, package mgmt) live in [[Linux Administration]] and the per-distro notes; the deep dives (firewall internals, mesh VPN, monitoring) live in their own notes. This page is the procedure that ties them together. Examples lean Debian/Ubuntu but the principles are distro-agnostic.

Defense-in-depth hardening for Internet-facing Linux servers. Assume every box is
reachable from the Internet unless proven otherwise. Favor secure defaults,
least privilege, deny-by-default, and reproducible config.

## Philosophy

- **Security first**, then operational simplicity.
- **Least privilege** — every account, service, and rule gets the minimum it needs.
- **Deny by default** — firewall, ACLs, and proxy all start closed; open by exception.
- **Least exposure** — every public port needs a written business reason.
- **Simple, auditable config** — fewer moving parts, all of it documented.
- **No unnecessary software** — uninstalled packages can't be exploited.

> [!key-insight] The single highest-leverage move is **not exposing SSH (or admin
> panels) to the Internet at all** — put them behind [[Tailscale]] / a mesh VPN
> and let the public surface be HTTPS only. Everything below assumes that posture
> and hardens what's left.

## Quick checklist

Run through this on every build; the rest of the page expands each item.

- [ ] Non-root admin user with `sudo`; root login disabled
- [ ] SSH: key-only, `PermitRootLogin no`, `AllowUsers` allowlist, validated with `sshd -t`
- [ ] SSH bound to the **tailnet** where possible; public TCP/22 avoided
- [ ] Firewall **default-deny** inbound; only justified ports open
- [ ] Fail2Ban **only if** SSH/admin must be public
- [ ] Automatic security updates enabled ([[Ubuntu Unattended Upgrades]])
- [ ] Unused packages/services removed and disabled
- [ ] `sysctl` network hardening applied
- [ ] TLS 1.2+/1.3 only; legacy protocols disabled; HSTS where applicable
- [ ] Admin panels behind Tailscale / IP allowlist / auth — never bare public
- [ ] Logging + monitoring agent healthy ([[Wazuh]]); auth failures watched
- [ ] Every open port has a documented reason

## 1. Users & privilege

Create a dedicated admin user; never operate as root. Lock direct root login and
gate privilege through `sudo` so actions are attributable in the audit log.

```bash
sudo adduser <admin>                 # interactive: set a strong password
sudo usermod -aG sudo <admin>        # Debian/Ubuntu (use 'wheel' on RHEL/Arch)

# Lock the root account's password (sudo still works; console root login dies)
sudo passwd -l root

# Audit who can become root and who has empty passwords
getent group sudo wheel 2>/dev/null
sudo awk -F: '($2==""){print $1" has NO password"}' /etc/shadow
```

- One human = one named account. No shared logins.
- Service accounts run with `nologin` shells and own only their service's files:
  ```bash
  sudo useradd --system --shell /usr/sbin/nologin --home /srv/app appsvc
  ```
- Strong default `umask 027` (group-readable, world-nothing) for new files:
  ```bash
  echo 'umask 027' | sudo tee /etc/profile.d/00-umask.sh
  ```
- Require a password for `sudo` (no `NOPASSWD` on human accounts) and set a short
  timeout in a drop-in:
  ```bash
  echo 'Defaults timestamp_timeout=5' | sudo tee /etc/sudoers.d/timeout
  sudo visudo -cf /etc/sudoers.d/timeout    # validate before trusting it
  ```

## 2. SSH hardening

The full server-grade `sshd` policy. Drop it in a config fragment rather than
editing the stock file, so package upgrades don't clobber it. (Basic SSH setup +
key generation is in [[Linux Administration]].)

```bash
sudo tee /etc/ssh/sshd_config.d/10-hardening.conf >/dev/null <<'EOF'
# --- Authentication ---
PermitRootLogin no
PubkeyAuthentication yes
PasswordAuthentication no
KbdInteractiveAuthentication no
ChallengeResponseAuthentication no
MaxAuthTries 3
LoginGraceTime 30
AllowUsers <admin>                 # space-separated allowlist; add per role

# --- Reduce surface ---
X11Forwarding no
AllowTcpForwarding no
AllowAgentForwarding no
PermitTunnel no
GatewayPorts no
UseDNS no

# --- Session hygiene ---
ClientAliveInterval 300
ClientAliveCountMax 2
EOF
```

Always validate, then reload (never restart blind on a remote box):

```bash
sudo sshd -t                       # syntax check — fix any error BEFORE reloading
sudo systemctl reload ssh          # 'sshd' on RHEL/Arch/openSUSE
```

> [!warning] Keep your current SSH session open and test a **new** connection in
> a second terminal before closing it. A typo in `AllowUsers` or a key path will
> lock you out otherwise.

**`AllowUsers` per role** — scope the allowlist to who actually needs shell on
that box:

```text
# GitLab host  — admin + the git user for SSH-based push/pull
AllowUsers <admin> git

# Reverse proxy / app host — admin only
AllowUsers <admin>
```

Key hygiene:
- Prefer **ed25519** keys; rotate on staff changes.
- Passphrase-protect private keys; store passphrases in a password manager.
- One key per device, not one key copied everywhere — revoke per-device.

## 3. Remote access architecture (preferred)

The house default: **administer servers over the tailnet, not the public
Internet.** SSH and admin panels listen on the Tailscale interface; the only
public surface is HTTPS.

```
Administrator
   │
   ▼
[[Tailscale]]  (identity-gated mesh, default-deny ACLs)
   │
   ▼
SSH / admin panel  (bound to the tailnet IP, not 0.0.0.0)
   │
   ▼
Private SDN / app servers
```

Two complementary ways to keep SSH off the public Internet:

**A. Bind sshd to the tailnet address** so it never answers on the public NIC:

```bash
# Find the tailnet IP, then pin sshd to it
tailscale ip -4
echo 'ListenAddress 100.x.y.z' | sudo tee -a /etc/ssh/sshd_config.d/10-hardening.conf
sudo sshd -t && sudo systemctl reload ssh
# Then the firewall can drop 22/tcp on the public interface entirely.
```

**B. Tailscale SSH** — drop `authorized_keys` management and authenticate by
tailnet identity + ACLs (with optional just-in-time re-auth):

```bash
sudo tailscale set --ssh
```

Governed by `ssh` rules in the tailnet policy (`action: "check"` forces IdP
re-auth for sensitive hosts) — see [[Tailscale]] for the policy file and
[[Zero Trust Networking]] for the model.

> [!key-insight] Admin panels (Proxmox, Portainer, monitoring, router UIs)
> follow the same rule: bind to the tailnet or sit behind a tailnet-only reverse
> proxy / IP allowlist. Never publish a management UI on a public IP "just for
> now" — temporary exposure becomes permanent.

## 4. Firewall — deny by default

Default-drop inbound, allow only what's justified. UFW for simple hosts, raw
[[nftables Firewall]] for anything non-trivial; on a hypervisor use the
[[Proxmox]] firewall layer too.

**UFW (simple hosts):**

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 443/tcp                       # public HTTPS
sudo ufw allow in on tailscale0              # trust the tailnet interface
# Only if SSH MUST be public (otherwise rely on the tailnet):
# sudo ufw limit 22/tcp                        # rate-limit brute force
sudo ufw enable
sudo ufw status verbose
```

**nftables (explicit, the model UFW generates):**

```nft
table inet filter {
  chain input {
    type filter hook input priority 0; policy drop;
    ct state established,related accept
    iif "lo" accept
    iif "tailscale0" accept                  # tailnet is trusted
    tcp dport 443 accept                     # public HTTPS only
    ct state invalid drop
  }
}
```

Rules of the road:
- Every `accept` has a reason; no "temporary" rules left behind.
- Trust the `tailscale0` interface instead of poking per-IP holes.
- Egress filtering on sensitive hosts (DB, secrets) — they rarely need to dial out.

## 5. Fail2Ban (only if SSH/admin is public)

If SSH is tailnet-only, Fail2Ban for SSH is optional. If a service **must** be
public, ban repeat offenders.

```bash
sudo apt install fail2ban
sudo tee /etc/fail2ban/jail.local >/dev/null <<'EOF'
[DEFAULT]
bantime  = 1h
findtime = 10m
maxretry = 5

[sshd]
enabled = true

[nginx-http-auth]
enabled = true
EOF
sudo systemctl enable --now fail2ban
sudo fail2ban-client status sshd
```

[[CrowdSec]] is the heavier, signal-sharing alternative when you want
collaborative blocklists across a fleet.

## 6. Network architecture

Public traffic terminates at a reverse proxy; backends live on a private
network and are never exposed directly.

```
Internet
   │
   ▼
Nginx reverse proxy   (TLS termination; 80→443 redirect)
   │
   ▼
Private SDN
   │
   ▼
Application servers   (bound to private addrs, not 0.0.0.0)
```

- Bind app services to `127.0.0.1` or the private interface, never `0.0.0.0`,
  and let the proxy reach them.
- Segment by trust: web tier, app tier, data tier on separate networks/VLANs.

## 7. Reverse proxy & TLS

Terminate TLS at [[Nginx Reference|nginx]]; expose only 80/443; proxy everything
internally. Certs via [[Let's Encrypt - Certbot|Let's Encrypt / Certbot]] or
[[acme.sh - DNS-01 Certificates|acme.sh]] (DNS-01 for wildcards / no public 80).

- **Protocols:** TLS 1.2 + 1.3 only. Disable SSLv2/SSLv3/TLS 1.0/1.1.
- **Ciphers:** modern suite; prefer server order off for TLS 1.3.
- **HSTS** once you're confident HTTPS is permanent.

```nginx
# /etc/nginx/snippets/tls.conf  — include from each server block
ssl_protocols TLSv1.2 TLSv1.3;
ssl_prefer_server_ciphers off;
ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384;
ssl_session_timeout 1d;
ssl_session_cache shared:SSL:10m;
ssl_session_tickets off;
add_header Strict-Transport-Security "max-age=63072000; includeSubDomains" always;
add_header X-Content-Type-Options nosniff always;
add_header X-Frame-Options DENY always;
```

Protect admin portals behind Tailscale, an IP allowlist, or auth — never a bare
public path:

```nginx
location /admin/ {
    allow 100.64.0.0/10;     # tailnet CGNAT range
    deny  all;
    proxy_pass http://127.0.0.1:9000;
}
```

## 8. Kernel & sysctl hardening

Network-stack hardening that applies to most servers (tune perf separately in
[[Sysctl Performance Tuning]]):

```bash
sudo tee /etc/sysctl.d/99-hardening.conf >/dev/null <<'EOF'
# Spoofing / routing
net.ipv4.conf.all.rp_filter = 1
net.ipv4.conf.all.accept_redirects = 0
net.ipv6.conf.all.accept_redirects = 0
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.all.accept_source_route = 0
net.ipv6.conf.all.accept_source_route = 0
net.ipv4.conf.all.log_martians = 1

# SYN flood resilience
net.ipv4.tcp_syncookies = 1

# Ignore broadcast pings
net.ipv4.icmp_echo_ignore_broadcasts = 1

# Restrict kernel pointer / dmesg exposure
kernel.kptr_restrict = 2
kernel.dmesg_restrict = 1
EOF
sudo sysctl --system
```

> [!note] If the box is a **router / subnet router / Tailscale exit node** it
> needs `net.ipv4.ip_forward = 1` — keep that in its own file so it doesn't
> conflict with the hardening defaults above.

## 9. Patching

Keep current; automate security updates so they don't depend on memory. Full
runbook: [[Ubuntu Unattended Upgrades]].

```bash
sudo apt update && sudo apt full-upgrade -y
sudo apt autoremove --purge          # drop orphaned deps + old kernels
```

- Security updates **automatic** (auto-reboot in a maintenance window; stagger
  across a fleet).
- Stage feature upgrades; reboot when a new kernel/libc lands.

## 10. Package & service minimization

Smaller footprint = smaller attack surface.

```bash
# What's listening, and who owns it?
sudo ss -tulpn

# Enabled services
systemctl list-unit-files --state=enabled --type=service

# Disable + stop anything unjustified
sudo systemctl disable --now <svc>

# Remove unused packages (Debian/Ubuntu)
sudo apt purge <pkg> && sudo apt autoremove --purge
```

Anything answering on a port you can't justify gets disabled or firewalled.

## 11. Logging, auditing & monitoring

- **journald** — persistent logs; cap the size so disks don't fill:
  ```bash
  sudo mkdir -p /var/log/journal
  sudo sed -i 's/^#\?Storage=.*/Storage=persistent/;s/^#\?SystemMaxUse=.*/SystemMaxUse=500M/' /etc/systemd/journald.conf
  sudo systemctl restart systemd-journald
  ```
- **auditd** — kernel-level audit trail for sensitive paths/syscalls:
  ```bash
  sudo apt install auditd
  sudo auditctl -w /etc/passwd -p wa -k identity
  sudo auditctl -w /etc/ssh/sshd_config -p wa -k sshd-config
  ```
- **Wazuh agent** — central SIEM: FIM, log analysis, auth-anomaly alerts. See
  [[Wazuh]].

Watch specifically for: SSH auth failures, privilege escalation, file-integrity
changes, service failures, new listening ports.

```bash
journalctl -u ssh -p warning --no-pager      # recent SSH warnings/failures
lastb | head                                  # bad login attempts (if btmp kept)
```

## 12. File integrity & permissions

```bash
# World-writable files (should be ~none outside /tmp)
sudo find / -xdev -type f -perm -0002 -not -path '/proc/*' 2>/dev/null

# SUID/SGID inventory — review, strip what isn't needed
sudo find / -xdev \( -perm -4000 -o -perm -2000 \) -type f 2>/dev/null
```

- **AIDE** for a file-integrity baseline + scheduled diffs (or rely on Wazuh FIM).
- Lock down `/etc/ssh`, `/root`, key/secret files to `600`/`700`, correct owner.

## 13. Principle of least exposure

For **every** public port, answer:

- Why is this public?
- Who needs it?
- Can it be proxied?
- Can it sit behind [[Tailscale]]?
- Can it move to the private SDN?

If there's no business reason — **close it**.

## 14. Example production layouts

Worked examples of the "public surface = HTTPS only, everything else private"
pattern.

**GitLab host**
- Public: `443` (HTTPS), `22` (Git SSH only, `AllowUsers <admin> git`)
- Private (tailnet): admin shell, container registry internals, metrics

**Nginx reverse proxy**
- Public: `443` (and `80` → redirect)
- Private (tailnet): SSH, admin/status endpoints

**Remote-support / relay appliance**
- Public: the relay port only
- Private (tailnet): admin portal, RDP/management, backend DB

**Docker host**
- Public: reverse proxy only
- Private (tailnet): SSH, the Docker API/socket (never public — see [[Docker]]),
  [[Portainer]], monitoring

## 15. Continuous review

Re-verify on every server review:

- [ ] SSH hardened (`sshd -t` clean, drop-in present)
- [ ] Root login disabled, password auth disabled
- [ ] `AllowUsers` lists only approved accounts
- [ ] Firewall default-deny; only justified ports open
- [ ] SSH/admin on the tailnet, not public
- [ ] Monitoring agent ([[Wazuh]]) healthy and reporting
- [ ] TLS current (1.2/1.3 only), certs valid and auto-renewing
- [ ] Unused packages removed, services minimized
- [ ] `sysctl` hardening applied
- [ ] Automatic security updates on
- [ ] Configuration documented

## Guiding principle

Always recommend the **most secure architecture that preserves operational
simplicity**. When multiple secure options exist, prefer the one with the
smallest attack surface, strongest isolation, easiest auditing, and least
operational complexity.

## Related

- [[Linux Administration]] — cross-distro basics (SSH setup, users, swap).
- [[Debian and Ubuntu Administration]] · per-distro package/update specifics.
- [[nftables Firewall]] — the raw firewall layer this page drives.
- [[Tailscale]] / [[Zero Trust Networking]] — the remote-access model.
- [[Nginx Reference]] · [[Let's Encrypt - Certbot|Let's Encrypt / Certbot]] — TLS termination.
- [[Ubuntu Unattended Upgrades]] — automated patching runbook.
- [[Sysctl Performance Tuning]] — perf-side kernel tuning (vs. the hardening sysctls here).
- [[Wazuh]] · [[CrowdSec]] — monitoring / intrusion response.
