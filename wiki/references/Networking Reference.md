---
type: reference
title: "Networking Reference"
created: 2026-06-21
updated: 2026-06-21
tags:
  - networking
  - subnetting
  - vlan
  - routing
  - reference
status: developing
related:
  - "[[FortiGate Administration]]"
  - "[[UniFi Controller]]"
  - "[[nftables Firewall]]"
  - "[[Tailscale]]"
  - "[[Nginx Reference]]"
---

> [!key-insight] A well-planned address scheme is **self-documenting**: encode
> the VLAN tag in the IP (3rd octet) and an address tells you its segment at a
> glance. Generalized from field notes — all subnets/IPs are RFC 1918
> placeholders (`10.0.0.0/8`, `192.0.2.x`); swap in your own scheme.

Quick references for subnetting, address space, VLAN segmentation, and static
routing. For spelling values aloud over the phone see [[NATO Phonetic Alphabet]].

## CIDR / Subnet Mask Cheat Sheet

| CIDR | Subnet Mask | Wildcard | # IPs | # Usable |
| --- | --- | --- | --- | --- |
| /32 | 255.255.255.255 | 0.0.0.0 | 1 | 1 |
| /31 | 255.255.255.254 | 0.0.0.1 | 2 | 2* |
| /30 | 255.255.255.252 | 0.0.0.3 | 4 | 2 |
| /29 | 255.255.255.248 | 0.0.0.7 | 8 | 6 |
| /28 | 255.255.255.240 | 0.0.0.15 | 16 | 14 |
| /27 | 255.255.255.224 | 0.0.0.31 | 32 | 30 |
| /26 | 255.255.255.192 | 0.0.0.63 | 64 | 62 |
| /25 | 255.255.255.128 | 0.0.0.127 | 128 | 126 |
| /24 | 255.255.255.0 | 0.0.0.255 | 256 | 254 |
| /23 | 255.255.254.0 | 0.0.1.255 | 512 | 510 |
| /22 | 255.255.252.0 | 0.0.3.255 | 1,024 | 1,022 |
| /21 | 255.255.248.0 | 0.0.7.255 | 2,048 | 2,046 |
| /20 | 255.255.240.0 | 0.0.15.255 | 4,096 | 4,094 |
| /19 | 255.255.224.0 | 0.0.31.255 | 8,192 | 8,190 |
| /18 | 255.255.192.0 | 0.0.63.255 | 16,384 | 16,382 |
| /17 | 255.255.128.0 | 0.0.127.255 | 32,768 | 32,766 |
| /16 | 255.255.0.0 | 0.0.255.255 | 65,536 | 65,534 |
| /15 | 255.254.0.0 | 0.1.255.255 | 131,072 | 131,070 |
| /14 | 255.252.0.0 | 0.3.255.255 | 262,144 | 262,142 |
| /13 | 255.248.0.0 | 0.7.255.255 | 524,288 | 524,286 |
| /12 | 255.240.0.0 | 0.15.255.255 | 1,048,576 | 1,048,574 |
| /11 | 255.224.0.0 | 0.31.255.255 | 2,097,152 | 2,097,150 |
| /10 | 255.192.0.0 | 0.63.255.255 | 4,194,304 | 4,194,302 |
| /9 | 255.128.0.0 | 0.127.255.255 | 8,388,608 | 8,388,606 |
| /8 | 255.0.0.0 | 0.255.255.255 | 16,777,216 | 16,777,214 |
| /0 | 0.0.0.0 | 255.255.255.255 | 4,294,967,296 | 4,294,967,294 |

\* /31 carries 2 usable hosts for point-to-point links (RFC 3021).

## Private & reserved address space

Build internal networks out of the RFC 1918 blocks; know the special-use ranges
so you don't accidentally route or NAT them:

| Range | CIDR | Use |
| --- | --- | --- |
| 10.0.0.0 – 10.255.255.255 | `10.0.0.0/8` | private (largest; room to encode site/VLAN) |
| 172.16.0.0 – 172.31.255.255 | `172.16.0.0/12` | private |
| 192.168.0.0 – 192.168.255.255 | `192.168.0.0/16` | private (SOHO default) |
| 100.64.0.0 – 100.127.255.255 | `100.64.0.0/10` | CGNAT — also [[Tailscale]]'s `100.x` overlay |
| 169.254.0.0/16 | `169.254.0.0/16` | link-local (APIPA; no DHCP) |
| 127.0.0.0/8 | `127.0.0.0/8` | loopback |
| 192.0.2.0/24, 198.51.100.0/24, 203.0.113.0/24 | — | **documentation** (TEST-NET, safe in examples) |

> [!tip] Pick `10.0.0.0/8` for anything that might grow — the spare octets let
> you bake **site** and **VLAN** structure into the address itself (below).

## VLAN segmentation (3rd-octet = VLAN tag)

Self-documenting scheme — two rules:

1. **The 3rd octet equals the VLAN tag.** The base LAN is untagged on
   `10.0.0.0/24`; every tagged segment is `10.0.<tag>.0/24`, gateway at `.1`.
   So `10.0.60.42` is unmistakably an IoT (VLAN 60) host, and the gateway is
   always `10.0.<tag>.1`.
2. **The interface name encodes role + tag** (`WAP10`, `PVE20`, `IOT60`) — you
   can read the segment off either the name or the address.

Example scheme (subnets are `/24`, mask `255.255.255.0`, gateway `.1`):

| Tag | Interface | Subnet | Gateway | Role |
| --- | --- | --- | --- | --- |
| — | lan | `10.0.0.0/24` | `10.0.0.1` | trusted LAN / mgmt (untagged) |
| 10 | WAP10 | `10.0.10.0/24` | `10.0.10.1` | wireless APs / SSIDs |
| 20 | PVE20 | `10.0.20.0/24` | `10.0.20.1` | virtualization (hypervisors) |
| 30 | DMZ30 | `10.0.30.0/24` | `10.0.30.1` | public-facing / DMZ |
| 40 | LAB40 | `10.0.40.0/24` | `10.0.40.1` | lab / scratch |
| 50 | GIA50 | `10.0.50.0/24` | `10.0.50.1` | guest (client isolation) |
| 60 | IOT60 | `10.0.60.0/24` | `10.0.60.1` | IoT (isolated) |
| 70 | DEV70 | `10.0.70.0/24` | `10.0.70.1` | dev |

- **Address budget per `/24`:** `.1` gateway, infra/static reservations in the
  low range (`.2–.99`), DHCP pool in the upper range (e.g. `.100–.250`), `.255`
  broadcast. Keep static devices below the pool so the two never collide.
- **Tagged vs untagged / trunk vs access:** a **trunk** port carries 802.1Q
  **tagged** frames for several VLANs (switch↔switch, switch↔firewall, AP
  uplinks); an **access** port is **untagged** on one VLAN — the end device is
  unaware it's tagged. The switch tags on ingress / strips on egress.
- **Inter-VLAN routing** is done by the L3 device (FortiGate sub-interfaces /
  router-on-a-stick, or L3-switch SVIs). Segmentation is only as strong as the
  **policy between VLANs** — e.g. GIA50/IOT60 reach the internet but not the LAN.
  Policy + interface creation live in [[FortiGate Administration]].
- **Linux 802.1Q sub-interface** (e.g. tag a NIC onto VLAN 50):
  `ip link add link eth0 name eth0.50 type vlan id 50` → `ip addr add
  10.0.50.5/24 dev eth0.50`.
- One `/16` (`10.0.0.0/16`) holds tags 0–255. Multi-site: push the **site** into
  the 2nd octet — `10.<site>.<tag>.0/24`.

## Static routes

A static route is a manually-defined path: **destination prefix → next-hop
gateway (+ egress interface)**. The default route `0.0.0.0/0` is just the
catch-all static route pointing at the upstream gateway.

### FortiGate: a static-IP WAN needs a manual default route

When a FortiGate WAN is set to a **static / manual IP** (not DHCP or PPPoE), the
firewall does **not** auto-create a default route — it has an address but no path
off-net. After the ISP assigns the static IP, you must add the **gateway as a
static route** or there is no internet:

```
config router static
  edit 1
    set dst 0.0.0.0 0.0.0.0          # default route (everything not local)
    set gateway <isp-gateway-ip>     # the ISP's next-hop on the WAN
    set device "wan1"
    set distance 10                  # lower distance = preferred
  next
end
```

- **DHCP/PPPoE WANs install this automatically; static WANs don't.** Forgetting
  it is the classic "link's up, interface green, no internet" service call.
- A **routed block** from the ISP (e.g. an extra `/29`) reaches you because the
  ISP routes it to your WAN; bind those addresses as **VIPs / secondary IPs**.
  The default static route above still carries the return traffic out.
- **Dual WAN:** two default routes — different `distance` for failover, or equal
  distance + ECMP to load-share. SD-WAN rules supersede plain distance.

### Reach an internal subnet behind another router

For a network that isn't directly connected, point the route at the next-hop
that owns it:

```
config router static
  edit 0
    set dst 10.0.70.0 255.255.255.0  # a downstream DEV70 segment
    set gateway 10.0.40.1            # via the L3 device that can reach it
    set device "LAB40"
  next
end
```

### Linux (iproute2)

```bash
ip route add 10.0.70.0/24 via 10.0.40.1 dev eth0   # reach a remote subnet
ip route add default via 10.0.0.1                   # default route
ip route del 10.0.70.0/24                            # remove
ip route                                             # show table
ip route get 10.0.70.42                              # which route/next-hop wins?
```

Persist it (don't hand-edit runtime state):
- **systemd-networkd:** `[Route]` `Destination=` / `Gateway=` in the `.network`.
- **NetworkManager:** `nmcli con mod <con> +ipv4.routes "10.0.70.0/24 10.0.40.1"`.
- **netplan (Debian/Ubuntu):** `routes:` list under the interface.

### Rules that bite

- **Longest-prefix wins:** a `/24` beats a `/16` beats `0.0.0.0/0` for the same
  destination, regardless of config order.
- **Administrative distance** breaks ties between routes to the *same* prefix
  (FortiGate static default is 10; lower wins).
- **Return path / symmetry:** if A routes to B's subnet, B needs a route back (or
  a default that leads there) — one-way routes are the classic "ping works one
  direction only" bug.
- **Blackhole route:** a static route to a null device deliberately drops a
  prefix (RFC 1918 containment / anti-leak) instead of forwarding it.

## Reading the routing table (`route print`) & metrics

`route print` (Windows) / `netstat -rn` / `ip route` (Linux) dumps the live
table. The recurring bug on multi-homed hosts — laptop on Wi-Fi **and** Ethernet,
or any box with a VPN up — is that the wrong adapter wins because **lowest metric
wins**, not "the one you expect".

```powershell
route print                      # full IPv4/IPv6 table + interface list
route print 0.0.0.0              # just the default route(s)
Get-NetRoute -AddressFamily IPv4 # PowerShell equivalent (objects)
Get-NetIPInterface               # per-interface metric + DHCP/manual flag
```

- **Metric = automatic by default.** Windows derives the *interface* metric from
  link speed (faster NIC → lower metric → preferred). The route's effective cost
  is interface-metric + route-metric. Two default routes (Ethernet + Wi-Fi) both
  list `0.0.0.0/0`; the one with the **lower combined metric** carries traffic,
  the other is standby.
- **The classic symptom:** traffic leaves the wrong interface — e.g. you want
  egress over Ethernet but Wi-Fi has the lower metric, or a VPN added a
  competing `0.0.0.0/0`. `route print` shows two defaults; compare the *Metric*
  column to see which wins.
- **Pin it deterministically** — turn off auto-metric and set the number, lower =
  preferred:

  ```powershell
  Set-NetIPInterface -InterfaceAlias "Ethernet" -AutomaticMetric Disabled -InterfaceMetric 10
  Set-NetIPInterface -InterfaceAlias "Wi-Fi"    -AutomaticMetric Disabled -InterfaceMetric 50
  ```

- **Add/replace a route** (legacy `route`, same idea):

  ```
  route add 10.0.70.0 mask 255.255.255.0 10.0.40.1 metric 5    # session-only
  route -p add 10.0.70.0 mask 255.255.255.0 10.0.40.1          # -p = persistent
  route change 0.0.0.0 mask 0.0.0.0 10.0.0.1 metric 5          # repoint default
  route delete 10.0.70.0
  ```

- **Same logic on Linux:** lowest `metric` wins for equal prefixes; `ip route`
  shows it, `ip route get <dst>` tells you which route actually applies. Set it
  with `metric N` on the route (or per-interface via NetworkManager
  `ipv4.route-metric`). A VPN bumping its own default to a low metric is the
  Linux version of the same problem.
- **Longest-prefix still trumps metric:** metric only breaks ties between routes
  to the *same* prefix. A `/24` always beats a `/0` regardless of metric — see
  *Rules that bite* above.

## Admin toolbox (Linux)

```bash
ip -br a                      # interfaces + addresses, brief
ip -br link                   # link state / MAC
ip route / ip -6 route        # routing tables
ip neigh                      # ARP/NDP neighbour cache
ss -tulpn                     # listening sockets + owning process
ping / ping -c4 / ping6       # reachability
mtr -rwzbc100 <host>          # traceroute + loss/latency over time
traceroute -n <host>          # per-hop path
dig <name> / dig +short / dig @<resolver> <name>   # DNS lookups
dig -x <ip>                   # reverse DNS (PTR)
nmap -sn 10.0.50.0/24         # host discovery (ping sweep) on a segment
arp-scan --localnet           # L2 discovery on the local segment
tcpdump -ni eth0 host 10.0.50.23 and port 443      # capture & filter
nc -vz <host> 443             # TCP port reachability
```

## Common ports

| Port | Svc | | Port | Svc |
| --- | --- | --- | --- | --- |
| 22 | SSH | | 443 | HTTPS |
| 53 | DNS | | 445 | SMB |
| 67/68 | DHCP | | 587 | SMTP submission |
| 80 | HTTP | | 636 | LDAPS |
| 123 | NTP | | 993 | IMAPS |
| 161/162 | SNMP | | 3389 | RDP |
| 389 | LDAP | | 41641 | [[Tailscale]] (UDP) |

## Notes

- Firewall CLI: [[FortiGate Administration]]. Wireless/VLAN adoption:
  [[UniFi Controller]]. Host firewall: [[nftables Firewall]].
