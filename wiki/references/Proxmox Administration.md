---
type: reference
title: "Proxmox Administration"
created: 2026-06-21
updated: 2026-06-21
tags:
  - proxmox
  - virtualization
  - linux
status: developing
related:
  - "[[UniFi Controller]]"
  - "[[Let's Encrypt - Certbot|Let's Encrypt / Certbot]]"
  - "[[Linux Administration]]"
---

> [!key-insight] Generalized from field notes; host/client-specific values are placeholders.

Operational runbook for Proxmox VE: container resizing, certificate regeneration, and major-version upgrades.

## Resize a Container's Root Disk

Grow an LXC container's root filesystem online:

```bash
pct resize <ctid> rootfs +10G
```

## Regenerate Self-Signed Certificates

Use when the cluster's self-signed PKI is broken or needs rotation. Replace `<node>` with the actual node name.

```bash
rm /etc/pve/pve-root-ca.pem
rm /etc/pve/priv/pve-root-ca.key
rm /etc/pve/nodes/<node>/pve-ssl.pem
rm /etc/pve/nodes/<node>/pve-ssl.key
```

After removal, regenerate and reboot:

```bash
pvecm updatecerts -f
```

## Major-Version Upgrades

Proxmox ships a pre-flight checker for each major jump. Always run it first and resolve every warning before proceeding.

### PVE 7 to 8

```bash
pve7to8 --full
```

### PVE 8 to 9 (Bookworm to Trixie)

```bash
# 1. Pre-flight check
pve8to9 --full

# 2. Switch APT sources to Trixie + PVE 9 (no-subscription)
sed -i 's/bookworm/trixie/g' /etc/apt/sources.list
sed -i 's/bookworm/trixie/g' /etc/apt/sources.list.d/pve-enterprise.list 2>/dev/null || true
echo "deb http://download.proxmox.com/debian/pve trixie pve-no-subscription" \
  > /etc/apt/sources.list.d/pve-no-sub.list

# Optional: Ceph sources (replace 'quincy' with the target release, e.g. 'reef')
# echo "deb http://download.proxmox.com/debian/ceph-quincy trixie no-subscription" \
#   > /etc/apt/sources.list.d/ceph.list

# 3. Update + upgrade
apt update && apt full-upgrade -y

# 4. Reboot into Proxmox VE 9
reboot
```

## Notes

- For TLS certificates fronting the web UI, see [[Let's Encrypt - Certbot|Let's Encrypt / Certbot]].
- macOS guests require a special board key; see [[macOS Virtualization]].
