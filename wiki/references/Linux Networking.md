---
type: reference
title: Linux Networking
created: 2026-06-28
updated: 2026-06-28
tags:
  - linux
  - networking
  - bridge
  - bond
  - vlan
  - proxmox
  - sysadmin
status: developing
related:
  - "[[Networking Reference]]"
  - "[[Proxmox]]"
  - "[[nftables Firewall]]"
  - "[[VFIO GPU Passthrough]]"
  - "[[Debian and Ubuntu Administration]]"
  - "[[Linux Administration]]"
---

> [!key-insight] The **host-side virtual-networking** layer — bridges, bonds, and 802.1Q VLANs — the plumbing that connects VMs/containers to physical NICs. This is the L2 layer; the L3 side (CIDR, subnet scheme, static routes, routing-table debugging) lives in [[Networking Reference]]. Examples use RFC 1918 / TEST-NET placeholders.

A Linux **bridge** is a software switch in the kernel: it joins physical NICs and
virtual interfaces (VM taps, container veths) into one L2 segment. A **bond**
aggregates NICs for redundancy/throughput. **802.1Q VLANs** tag frames so one
wire carries many segments. Hypervisors ([[Proxmox]], libvirt) lean on all three.

## Runtime vs persistent

Two layers, don't confuse them:

- **Runtime** (`ip`, `bridge`) — changes the live kernel state; **lost on reboot**.
  Great for testing, never for production config.
- **Persistent** — a config renderer writes the interfaces at boot. Which one
  depends on the distro:

| Renderer | Where | Used by |
|----------|-------|---------|
| **ifupdown2** | `/etc/network/interfaces` | [[Proxmox]], Debian (classic) |
| **systemd-networkd** | `/etc/systemd/network/*.network`/`.netdev` | servers, Arch |
| **netplan** | `/etc/netplan/*.yaml` → backend | Ubuntu (see [[Debian and Ubuntu Administration]]) |
| **NetworkManager** | `nmcli` / keyfiles | desktops, RHEL/Fedora |

> [!warning] Pick **one** manager per host. Running netplan/NetworkManager and
> hand-edited `/etc/network/interfaces` at the same time fights over the same
> interfaces and produces nondeterministic boots.

## Interfaces — the iproute2 basics

```bash
ip -br link                       # link state + MACs, brief
ip -br addr                       # addresses, brief
ip link set enp1s0 up             # bring an interface up/down
ip addr add 10.0.0.5/24 dev enp1s0
ip addr flush dev enp1s0          # clear addresses
```

> [!note] Predictable interface names (`enp7s0f1` = PCI bus 7, slot 0, function
> 1) replace the old `eth0`. A multi-port card shows as `enpXs0f0..f3`; the
> onboard NIC is usually its own bus (`enp9s0`).

## Bridges

### Create at runtime (test only)

```bash
ip link add name br0 type bridge
ip link set enp1s0 master br0     # enslave a physical NIC
ip link set br0 up
ip link set enp1s0 up
bridge link                       # show bridge membership
bridge vlan show                  # VLAN filtering table (vlan-aware bridges)
```

### Persistent — ifupdown2 (`/etc/network/interfaces`)

This is the Proxmox/Debian format. The **physical port is `manual`** (no IP); the
bridge carries the address:

```ini
auto lo
iface lo inet loopback

auto enp9s0
iface enp9s0 inet manual

auto vmbr0
iface vmbr0 inet static
    address 10.0.0.5/24
    gateway 10.0.0.1
    bridge-ports enp9s0
    bridge-stp off            # spanning tree off for a simple host bridge
    bridge-fd 0               # forwarding delay 0 — no learning pause
```

Apply without a reboot (ifupdown2 supports reload):

```bash
ifreload -a                  # ifupdown2 (Proxmox) — apply /etc/network/interfaces
# classic ifupdown: ifdown vmbr0 && ifup vmbr0
```

### Persistent — systemd-networkd

```ini
# /etc/systemd/network/br0.netdev
[NetDev]
Name=br0
Kind=bridge
```
```ini
# /etc/systemd/network/br0.network
[Match]
Name=br0
[Network]
Address=10.0.0.5/24
Gateway=10.0.0.1
```
```ini
# /etc/systemd/network/uplink.network — enslave the NIC
[Match]
Name=enp1s0
[Network]
Bridge=br0
```
```bash
sudo systemctl restart systemd-networkd
networkctl status br0
```

## Bonds (link aggregation)

Aggregate NICs for redundancy or throughput. The common modes:

| Mode | Name | Needs switch config | Use |
|------|------|---------------------|-----|
| `active-backup` (1) | failover | no | simplest HA; one link active |
| `802.3ad` (4) | LACP | **yes** (LAG) | throughput + HA, negotiated |
| `balance-alb` (6) | adaptive LB | no | LB without switch support |

ifupdown2 bond feeding a bridge (the usual hypervisor pattern):

```ini
auto bond0
iface bond0 inet manual
    bond-slaves enp8s0f0 enp8s0f1
    bond-mode 802.3ad
    bond-miimon 100
    bond-xmit-hash-policy layer3+4

auto vmbr0
iface vmbr0 inet static
    address 10.0.0.5/24
    gateway 10.0.0.1
    bridge-ports bond0       # bridge sits on top of the bond
    bridge-stp off
    bridge-fd 0
```

> [!warning] `802.3ad` (LACP) **requires** a matching LAG on the switch. Without
> it the link flaps or blackholes. Use `active-backup` when you don't control the
> switch.

## VLANs (802.1Q)

### A single VLAN sub-interface

Tag one NIC onto a VLAN — `dev.tag` naming:

```bash
ip link add link enp1s0 name enp1s0.50 type vlan id 50
ip addr add 10.0.50.5/24 dev enp1s0.50
```

ifupdown2 equivalent:

```ini
auto enp1s0.50
iface enp1s0.50 inet static
    address 10.0.50.5/24
```

### Bridge pinned to one VLAN (segment a whole bridge)

Put a VLAN sub-interface in as the bridge port, so **everything** on that bridge
lives on VLAN 50 — clean per-segment bridges:

```ini
auto vmbr1
iface vmbr1 inet manual
    bridge-ports enp8s0f1.50    # the .50 tags the whole bridge onto VLAN 50
    bridge-stp off
    bridge-fd 0
```

### VLAN-aware bridge (trunk + per-port tagging)

One bridge carries **all** VLANs as a trunk; each VM/CT picks its VLAN with a
`tag=`. This is the flexible default on a hypervisor — no separate bridge per
VLAN.

```ini
auto vmbr1
iface vmbr1 inet manual
    bridge-ports enp8s0f1
    bridge-stp off
    bridge-fd 0
    bridge-vlan-aware yes
    bridge-vids 2-4094          # which VLAN IDs the trunk allows
```

```bash
bridge vlan show               # inspect the per-port VLAN table
```

> [!key-insight] **VLAN-aware bridge vs per-VLAN bridge.** A *VLAN-aware* bridge
> is a trunk: one bridge, tag the VM (`tag=20`) to drop it on a segment — fewer
> bridges, switch-like behavior. A *pinned* bridge (`enpXsY.50`) is an access
> port baked into the bridge: simpler mental model, but you need one bridge per
> VLAN. Pick VLAN-aware when VMs span many segments; pin when a bridge serves
> exactly one.

## veth & namespaces (containers)

A **veth** pair is a virtual cable: one end in a namespace/container, the other
on a bridge. This is how containers attach to host networking under the hood.

```bash
ip link add veth0 type veth peer name veth0-br
ip link set veth0-br master br0      # host end onto the bridge
# veth0 goes into the container/namespace
```

LXC/Docker manage this for you; you rarely build veths by hand outside debugging.

## VXLAN (L2 overlay over L3)

VXLAN tunnels an L2 segment across a routed L3 network — a **VNI** (VXLAN Network
Identifier, the 24-bit "VLAN tag" allowing ~16M segments) stretches one broadcast
domain over many hosts. This is how multi-host overlays work ([[Proxmox]] SDN
VXLAN zones, Docker overlay networks).

```bash
# Unicast/FDB VXLAN over the underlay NIC, UDP 4789, VNI 100
ip link add vxlan100 type vxlan id 100 dev enp1s0 dstport 4789 \
    local 10.0.0.5 nolearning
ip link set vxlan100 master br0        # drop it on a bridge like any other port
ip link set vxlan100 up

# Tell it about a remote VTEP (the other host's underlay IP) for a MAC:
bridge fdb append 00:00:00:00:00:00 dev vxlan100 dst 10.0.0.6
```

| Term | Meaning |
|------|---------|
| **VNI** | 24-bit segment ID (the overlay's "VLAN") |
| **VTEP** | VXLAN Tunnel Endpoint — the host that encaps/decaps (each Proxmox node) |
| **Underlay** | the routed network carrying the tunnels |
| **Overlay** | the virtual L2 the guests see |

> [!key-insight] **VLAN vs VXLAN.** A VLAN (802.1Q) is local L2 — 4094 tags, must
> be trunked hop-by-hop across switches. VXLAN rides **on top of IP**, so two
> hosts on different subnets/sites share one L2 segment without the physical
> switches knowing — at the cost of **~50 bytes of encap overhead** (raise the
> underlay MTU to ~1550+, or jumbo, to avoid fragmenting guest 1500-byte frames).
> Reach for VXLAN for cross-node/cross-site overlays; plain VLANs for one L2 fabric.

For EVPN (BGP-distributed MAC/IP instead of multicast/static FDB) and a managed
control plane, use [[Proxmox]] SDN's VXLAN/EVPN zones rather than wiring this by
hand.

## Proxmox bridges (worked example)

Proxmox uses ifupdown2 (`/etc/network/interfaces`); the GUI (Datacenter → node →
System → **Network**) writes the same file. Bridges are named `vmbrN`.

Example host: a **4-port NIC** (`enp7s0f0/f1`, `enp8s0f0/f1`), the **onboard
NIC** (`enp9s0`), and a **WiFi adapter** (passed through, see [[Proxmox]]):

| Bridge | Port | VLAN-aware | Role |
|--------|------|------------|------|
| `vmbr0` | `enp9s0` (onboard) | no | **PVE management** — 10.0.0.5/24, the node's own IP |
| `vmbr1` | `enp8s0f1` | yes | VM/CT segment (trunk; tag per guest) |
| `vmbr2` | `enp7s0f1` | yes | second segment — e.g. **pfSense WAN** |
| `vmbr3` | `enp7s0f0` | yes | spare / additional segment |

> [!key-insight] Keep the **management bridge (`vmbr0`) on its own NIC** and out
> of the VLAN-aware trunks. If you lose the trunk config you can still reach the
> node to fix it. The onboard NIC for mgmt, the multi-port card for guest
> traffic, is the reliable split.

`/etc/network/interfaces` for that layout:

```ini
auto lo
iface lo inet loopback

# Onboard NIC → management
auto enp9s0
iface enp9s0 inet manual
auto vmbr0
iface vmbr0 inet static
    address 10.0.0.5/24
    gateway 10.0.0.1
    bridge-ports enp9s0
    bridge-stp off
    bridge-fd 0

# 4-port NIC → guest bridges (VLAN-aware trunks)
auto enp8s0f1
iface enp8s0f1 inet manual
auto vmbr1
iface vmbr1 inet manual
    bridge-ports enp8s0f1
    bridge-stp off
    bridge-fd 0
    bridge-vlan-aware yes
    bridge-vids 2-4094

auto enp7s0f1
iface enp7s0f1 inet manual
auto vmbr2
iface vmbr2 inet manual
    bridge-ports enp7s0f1
    bridge-stp off
    bridge-fd 0
    bridge-vlan-aware yes
    bridge-vids 2-4094
```

### Tagging a guest onto a VLAN

With a VLAN-aware bridge, set the `tag` on the guest's NIC — Proxmox handles the
802.1Q:

```bash
# VM: put it on VLAN 20 via vmbr1
qm set <vmid> --net0 virtio,bridge=vmbr1,tag=20

# LXC: same idea
pct set <ctid> --net0 name=eth0,bridge=vmbr1,ip=10.0.20.50/24,gw=10.0.20.1,tag=20
```

The guest sees a plain untagged NIC; the bridge tags on the way out. Map tags to
subnets with the [[Networking Reference|3rd-octet VLAN scheme]] (VLAN 20 →
`10.0.20.0/24`).

### Virtualized pfSense / OPNsense (multi-bridge router-VM)

A firewall VM needs **two** bridges — one facing the internet (WAN), one facing
the LAN. Give the VM a NIC on each:

```bash
# net0 = WAN (vmbr2, the port cabled to the modem/ISP handoff)
# net1 = LAN (vmbr1, the port cabled to the internal switch)
qm set <vmid> --net0 virtio,bridge=vmbr2      # WAN
qm set <vmid> --net1 virtio,bridge=vmbr1      # LAN
```

> [!key-insight] **`vmbr2` = WAN, `vmbr1` = LAN** in this build. The WAN bridge's
> physical port goes to the ISP handoff; nothing else should sit on that bridge.
> The firewall VM does the routing/NAT between the two bridges — the Proxmox host
> itself stays on `vmbr0` (mgmt) and is **not** in the traffic path. If the WAN
> segment is delivered tagged, either pin `vmbr2` to that VLAN or set the `tag`
> on the WAN NIC.

For passing a **physical** WAN port or a WiFi adapter straight into a guest
(bypassing the bridge entirely), use PCI/USB passthrough — see the WiFi-to-Kali
example in [[Proxmox]] and the mechanics in [[VFIO GPU Passthrough]] /
[[IOMMU and Device Groups]].

## Verify & debug

```bash
ip -br link                     # all links + state
bridge link                     # bridge memberships
bridge vlan show                # per-port VLAN table (vlan-aware bridges)
cat /proc/net/bonding/bond0     # bond mode + slave status
ip -d link show vmbr1           # detailed bridge flags (vlan_filtering, etc.)
```

- **VM has no network:** confirm the guest NIC's `bridge=` matches a real bridge
  and (for VLAN-aware) the `tag` is inside `bridge-vids`.
- **Host unreachable after edits:** you likely put the IP on the port instead of
  the bridge, or removed `vmbr0`'s address. Console in and `ifreload -a`.
- **Bond flapping:** LACP without a switch LAG — drop to `active-backup`.

## Notes

- L3 / addressing / routes / subnet scheme: [[Networking Reference]].
- Host firewall on top of these bridges: [[nftables Firewall]].
- Proxmox runbook (VMs, LXC, passthrough): [[Proxmox]].
- Ubuntu netplan specifics: [[Debian and Ubuntu Administration]].
