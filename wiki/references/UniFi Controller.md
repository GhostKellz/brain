---
type: reference
title: "UniFi Controller"
created: 2026-06-21
updated: 2026-06-21
tags:
  - unifi
  - networking
  - adoption
status: developing
related:
  - "[[Let's Encrypt - Certbot|Let's Encrypt / Certbot]]"
  - "[[FortiGate Administration]]"
  - "[[Networking Reference]]"
---

> [!key-insight] Generalized from field notes; host/client-specific values are placeholders.

Adopting UniFi devices to a self-hosted controller, including Layer 3 adoption patterns and TLS certs.

## Adopt a Device via `set-inform`

SSH into the device (default credentials or your provisioning credentials), then point it at the controller's inform endpoint. Replace `<controller-host>` with the controller's FQDN or IP.

```bash
set-inform https://<controller-host>:8080/inform
```

If you hit an `Unknown[12]` error, retry over plain HTTP first, then re-adopt:

```bash
set-inform http://<controller-host>:8080/inform
```

You can also use a controller IP directly:

```bash
set-inform http://<ip>:8080/inform
```

## Layer 3 Adoption (Remote Networks)

When devices and controller sit on different subnets, the device's Layer 2
discovery broadcast never reaches the controller, so it stays "pending" forever.
Two reliable ways to point it across the boundary — DHCP Option 43, or a local
`unifi` DNS record. Either works; pick whichever your environment already owns.

### Option A — DHCP Option 43 (preferred where the firewall is the DHCP server)

Option 43 is a DHCP vendor-specific option that delivers the controller IP in
the device's lease, so it informs the right place with no per-device config.
UniFi expects a sub-option TLV:

```
01 04 <4-byte controller IP in hex>
   │  │  └─ controller IP, one hex byte per octet
   │  └──── length = 4 (an IPv4 address)
   └─────── sub-option 0x01 = controller IP
```

Worked example — controller `192.168.1.2` → `C0 A8 01 02` → value
**`0104C0A80102`**.

We run **FortiGate** as the DHCP server, so the option goes on the FortiGate
scope that serves the device VLAN — full CLI recipe in
[[FortiGate Administration]]. The value is identical across vendors; only the
formatting (spaces / colons / `0x`) differs:

| Platform | Where | Type | Value for 192.168.1.2 |
|----------|-------|------|-----------------------|
| **FortiGate** | `config system dhcp server` → `config options` | hex | `0104C0A80102` |
| pfSense / OPNsense | Services → DHCPv4 → Option 43 | String | `0104C0A80102` |
| Sophos UTM | DHCP → vendor option | hex | `01:04:C0:A8:01:02` |
| MikroTik | `/ip dhcp-server option` | raw | `0x0104C0A80102` |
| Cisco IOS | `ip dhcp pool` → `option 43 hex` | hex | `0104.C0A8.0102` |
| Windows Server DHCP | Scope Options → 043 | binary | `01 04 C0 A8 01 02` |

> [!key-insight]
> The `01 04` prefix is the UniFi vendor header and is required. A device that
> gets a correct lease but never adopts almost always has a malformed option-43
> value (missing the `0104`, or an extra `2b`/`2b06` wrapper).

### Option B — local DNS `unifi` record (Windows Server DNS snap-in)

UniFi devices look up the hostname **`unifi`** by default. Publish an `A` record
for it on the DNS your devices use and they resolve straight to the controller —
no DHCP changes, and it survives DHCP server swaps.

On **Windows Server** (DNS Manager — `dnsmgmt.msc`):

1. Forward Lookup Zones → your internal zone (e.g. `<domain>`).
2. Right-click → **New Host (A or AAAA)…**
3. **Name:** `unifi`  ·  **IP address:** `<controller-ip>`  ·  tick *Create
   associated pointer (PTR)* if you keep reverse zones.
4. Devices in that domain now resolve `unifi.<domain>` → controller and adopt.

Equivalent record on any other DNS (BIND/dnsmasq/UniFi's own DNS):

```
unifi  IN  A  <controller-ip>
```

> [!key-insight]
> The device must use this DNS server and have the matching DNS **suffix**
> (`<domain>`) pushed via DHCP, or the bare `unifi` lookup won't resolve. This is
> why the firewall-DHCP Option 43 route is often simpler when one box owns both
> DHCP and the device VLAN.

## TLS Certificate (Let's Encrypt)

Issue a standalone cert for the controller hostname, then wire it into the controller / reverse proxy. Stop the web server on port 80 first so certbot can bind it.

```bash
sudo systemctl stop nginx
sudo certbot certonly --standalone -d unifi.<domain>
sudo nano /etc/nginx/sites-available/unifi
```

Full standalone issuance and nginx integration details: [[Let's Encrypt - Certbot|Let's Encrypt / Certbot]].

## Notes

- The controller is commonly hosted on a Linux VM or LXC; see [[Proxmox Administration]].
