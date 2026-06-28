---
type: reference
title: Proxmox
aliases:
  - "Proxmox Administration"
  - "references/Proxmox-Administration"
created: 2026-06-21
updated: 2026-06-28
tags:
  - proxmox
  - virtualization
  - linux
  - zfs
  - backup
status: developing
related:
  - "[[UniFi Controller]]"
  - "[[Let's Encrypt - Certbot|Let's Encrypt / Certbot]]"
  - "[[Linux Administration]]"
  - "[[Linux Networking]]"
  - "[[VFIO GPU Passthrough]]"
  - "[[ZFS]]"
  - "[[3-2-1 Backup Strategy]]"
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

## Networking — bridges & passthrough

Proxmox networking is Linux bridges over `/etc/network/interfaces` (ifupdown2);
the GUI writes the same file. The full bridge/bond/VLAN reference — including this
host's `vmbr0` (mgmt on the onboard NIC) + `vmbr1`–`vmbr3` (VLAN-aware trunks on
the 4-port NIC) and the **virtualized pfSense** `vmbr2`=WAN / `vmbr1`=LAN
pattern — lives in **[[Linux Networking]]**. Quick PVE-specific bits:

```bash
# Attach a guest to a bridge; tag it onto a VLAN (VLAN-aware bridge)
qm set <vmid> --net0 virtio,bridge=vmbr1,tag=20
pct set <ctid> --net0 name=eth0,bridge=vmbr1,ip=10.0.20.50/24,gw=10.0.20.1,tag=20

ifreload -a            # apply /etc/network/interfaces without a reboot
```

### PCI / USB passthrough — WiFi adapter → Kali VM

Passing a physical device straight into a guest (instead of bridging it) is the
right move for a WiFi adapter that needs **monitor mode / packet injection** in a
Kali VM — the guest gets the real radio, not a virtual NIC. Mechanics and the
IOMMU prerequisite are in [[VFIO GPU Passthrough]] / [[IOMMU and Device Groups]];
the Proxmox steps:

```bash
# 0. IOMMU must be on (GRUB: intel_iommu=on  OR  amd_iommu=on) — see IOMMU note.
#    Verify groups exist:
dmesg | grep -e DMAR -e IOMMU
```

**PCIe WiFi card** — find its address + IOMMU group, then assign:

```bash
lspci -nnk | grep -iA3 -e network -e wireless     # e.g. 03:00.0 + [vendor:device]
# Confirm it's in its own IOMMU group (sharing a group = pass the whole group):
for g in /sys/kernel/iommu_groups/*; do
  echo "group ${g##*/}:"; for d in "$g"/devices/*; do echo "  $(lspci -nns ${d##*/})"; done
done

qm set <vmid> -hostpci0 03:00,pcie=1              # whole function; pcie=1 for q35
```

**USB WiFi dongle** (common for Kali injection adapters) — simpler, no IOMMU
group concerns:

```bash
qm monitor <vmid>                                  # or use the GUI: Hardware → Add → USB Device
# or pin by vendor:product so it survives re-plugging:
qm set <vmid> -usb0 host=0bda:8812                 # <vendor>:<product> from lsusb
```

> [!key-insight] **PCIe vs USB passthrough.** A built-in/mini-PCIe card → PCI
> passthrough (needs IOMMU; pass the whole group if it isn't isolated). A USB
> dongle → USB passthrough (no IOMMU dependency, hot-pluggable). For Kali,
> pin USB devices by `vendor:product` not bus/port so a re-plug doesn't detach
> them. Use the `q35` machine type for PCIe passthrough.

## Software-Defined Networking (SDN)

SDN (Datacenter → **SDN**) abstracts networking **cluster-wide**: define a
*zone* + *VNet* once at the datacenter level and it materializes on every node —
no per-node `/etc/network/interfaces` editing. Config lives in `/etc/pve/sdn/`
(replicated across the cluster) and lands on each node on **Apply**. For the raw
Linux primitives underneath (bridges, VLANs, VXLAN), see [[Linux Networking]].

### The pieces

| Object | What it is |
|--------|-----------|
| **Zone** | the transport mechanism (how isolation is done) — see types below |
| **VNet** | a virtual network (becomes a bridge on each node); guests attach here |
| **Subnet** | an IP range on a VNet — gateway, SNAT, and DHCP options |
| **IPAM** | IP address management — built-in PVE IPAM (or phpIPAM/NetBox) |
| **Controller** | control plane for EVPN zones (FRR/BGP); not needed for Simple/VLAN/VXLAN |

| Zone type | Isolation | Use |
|-----------|-----------|-----|
| **Simple** | host-only bridge + NAT | isolated guest net on one node; quick labs |
| **VLAN** | 802.1Q on an existing bridge | standard segmentation over a trunked uplink |
| **QinQ** | stacked 802.1Q (S-tag + C-tag) | multi-tenant VLANs inside a VLAN |
| **VXLAN** | L2 overlay over L3 (VNI) | stretch a segment across nodes/subnets |
| **EVPN** | VXLAN + BGP control plane (FRR) | routed multi-node fabric with a VRF/exit nodes |

### Create a network (Simple zone + VNet + subnet)

GUI flow (Datacenter → SDN): **Zones → Add → Simple**, then **VNets → Add**
(pick the zone), then select the VNet → **Subnets → Add**. Then **SDN → Apply**.
The CLI equivalent:

```bash
# Zone
pvesh create /cluster/sdn/zones --zone simplezone --type simple

# VNet on that zone (alias becomes a bridge on every node)
pvesh create /cluster/sdn/vnets --vnet vnet10 --zone simplezone

# Subnet on the VNet: gateway + SNAT so guests reach the outside
pvesh create /cluster/sdn/vnets/vnet10/subnets \
  --subnet 10.0.20.0/24 --type subnet --gateway 10.0.20.1 --snat 1

pvesh set /cluster/sdn          # APPLY — writes config to every node
```

Attach a guest to the VNet exactly like a bridge:

```bash
qm set <vmid> --net0 virtio,bridge=vnet10
```

For a **VXLAN zone** (multi-node overlay), add the zone with the peer node IPs
and a VNet with a VNI:

```bash
pvesh create /cluster/sdn/zones --zone vxzone --type vxlan \
  --peers 10.0.0.5,10.0.0.6,10.0.0.7         # the nodes' underlay IPs (VTEPs)
pvesh create /cluster/sdn/vnets --vnet vxnet --zone vxzone --tag 100   # VNI 100
pvesh set /cluster/sdn
```

> [!key-insight] **VLAN zone vs your hand-rolled VLAN-aware bridge.** The bridge
> in [[Linux Networking]] is per-node; an SDN **VLAN zone** is the same idea
> declared **once for the cluster** and auto-applied everywhere — better when VMs
> migrate between nodes and you want the segment to exist identically on all of
> them. Single-node homelab: a plain VLAN-aware `vmbrX` is simpler. Cluster:
> prefer SDN.

### Built-in DHCP (dnsmasq) + IPAM

Since PVE 8.1, SDN subnets can hand out leases via a per-zone **dnsmasq**
instance backed by PVE IPAM — guests on a VNet get IPs automatically, no
external DHCP server. Enable per subnet by adding a **DHCP range** (and set the
zone's DHCP backend to `dnsmasq`):

```bash
# Set the zone to use dnsmasq for DHCP
pvesh set /cluster/sdn/zones/simplezone --dhcp dnsmasq

# Give the subnet a lease range (gateway already set above)
pvesh set /cluster/sdn/vnets/vnet10/subnets/10.0.20.0-24 \
  --dhcp-range start-address=10.0.20.100,end-address=10.0.20.200
pvesh set /cluster/sdn               # apply
```

What happens under the hood:
- Proxmox runs a **dnsmasq per zone**, bound to the VNet bridge, serving only its
  subnet's range — leases/reservations tracked in **PVE IPAM**.
- A guest with a DHCP NIC on `vnet10` boots and gets `10.0.20.100–200`, gateway
  `10.0.20.1`. Static mappings come from IPAM (assign the VM an IP and dnsmasq
  reserves it by MAC).
- Inspect: leases live under `/var/lib/pve/sdn/`; the dnsmasq services show up as
  `dnsmasq@<zone>` units.

```bash
systemctl status dnsmasq@simplezone
journalctl -u dnsmasq@simplezone -e --no-pager
```

> [!key-insight] SDN DHCP makes a VNet **self-contained**: gateway, NAT, and
> addressing all defined on the subnet object and replicated cluster-wide. It
> replaces standing up a separate DHCP server (or a [[Linux Networking|firewall
> VM]]) for isolated/lab segments — but for a production LAN you usually still
> want your real firewall/DHCP (FortiGate, pfSense) owning the scope.

## Cloud-Init VM Templates

A cloud-init "golden image" lets you clone a fully-provisioned VM in seconds.
Build it once from a distro cloud image, convert to a template, then clone.

```bash
# 1. Fetch a cloud image (Ubuntu shown)
wget -O /var/lib/vz/template/iso/noble-cloudimg.img \
  https://cloud-images.ubuntu.com/releases/noble/release/ubuntu-24.04-server-cloudimg-amd64.img

# 2. Create a shell VM and import the disk
qm create <vmid> --name <name> --memory 2048 --cores 2 \
  --net0 virtio,bridge=<bridge>
qm importdisk <vmid> /var/lib/vz/template/iso/noble-cloudimg.img <storage>
qm set <vmid> --scsihw virtio-scsi-pci --scsi0 <storage>:vm-<vmid>-disk-0

# 3. Add the cloud-init drive + serial console (cloud images log to serial)
qm set <vmid> --ide2 <storage>:cloudinit
qm set <vmid> --boot c --bootdisk scsi0
qm set <vmid> --serial0 socket --vga serial0

# 4. Seed identity / network (or use --cicustom, below)
qm set <vmid> --ciuser <user> --cipassword <password> \
  --sshkeys ~/.ssh/authorized_keys
qm set <vmid> --ipconfig0 ip=dhcp           # or ip=<ip>/<mask>,gw=<gateway>
qm set <vmid> --agent enabled=1

# 5. Convert to template, then clone
qm template <vmid>
qm clone <vmid> <newid> --name <new-name> --full
```

For richer provisioning, drop a `#cloud-config` YAML in
`/var/lib/vz/snippets/` and reference it:

```bash
qm set <vmid> --cicustom "user=/var/lib/vz/snippets/<name>.yaml"
qm cloudinit dump <vmid> user      # preview the rendered user-data
```

> [!key-insight]
> `--serial0 socket --vga serial0` is not optional for cloud images — they
> render boot/console output to the serial port. Without it `qm terminal` and
> the cloud-init console are blank and first-boot debugging is painful.

### Writing the `#cloud-config` (cloud-init proper)

The simple `--ciuser/--sshkeys/--ipconfig0` flags cover identity + network.
`--cicustom` lets you supply full **cloud-init** user-data — the cross-platform
first-boot provisioning standard (it's the same format AWS/Azure/GCP consume).
Cloud-init runs **once on first boot**, ordered by modules. The `#cloud-config`
header is mandatory:

```yaml
#cloud-config
# /var/lib/vz/snippets/web.yaml
hostname: web01
manage_etc_hosts: true

users:
  - name: deploy
    groups: [sudo]
    sudo: "ALL=(ALL) NOPASSWD:ALL"
    shell: /bin/bash
    ssh_authorized_keys:
      - ssh-ed25519 AAAA... you@host

package_update: true
package_upgrade: true
packages:
  - qemu-guest-agent
  - nginx

write_files:
  - path: /etc/motd
    content: "Provisioned by cloud-init\n"

runcmd:                      # shell commands, last, in order
  - systemctl enable --now qemu-guest-agent
  - systemctl enable --now nginx

power_state:                # reboot so upgrades/kernel take effect
  mode: reboot
  condition: true
```

Key modules: `users`/`ssh_authorized_keys` (identity), `packages`/`package_*`
(install), `write_files` (drop config), `runcmd` (arbitrary commands, runs last),
`power_state` (reboot). There are three data parts — **user-data** (the above),
**meta-data** (instance id/hostname), and **network-config**; on Proxmox the
`--ipconfig0`/`--nameserver` flags generate network-config for you, so `cicustom
user=` is usually all you hand-write.

```bash
# debug on the booted VM
cloud-init status --long          # done / running / error
cloud-init schema --system        # validate the merged config
journalctl -u cloud-init          # what ran
# to re-run during template testing (NOT in prod):
cloud-init clean --logs && reboot
```

> [!key-insight]
> Cloud-init is **first-boot only** — it keys off the instance-id, so cloning a
> template re-runs it on each new clone (good) but editing user-data on an
> already-booted VM does nothing until `cloud-init clean` + reboot. This is also
> the cloud-init that [[Terraform]]'s Proxmox provider sets when it stamps out VMs.

## LXC Containers

```bash
# Create an unprivileged container with nesting (needed for systemd/docker-in-CT)
pct create <ctid> <storage>:vztmpl/<template>.tar.zst \
  --hostname <name> --storage <target> --rootfs <target>:8 \
  --net0 name=eth0,bridge=<bridge>,ip=<ip>/<mask>,gw=<gateway>,tag=<vlan> \
  --unprivileged 1 --features nesting=1 \
  --cores 2 --memory 2048 --tags <tag>

pct start <ctid>
pct enter <ctid>                       # attach a shell
pct exec  <ctid> -- <command>          # run one command
pct stop  <ctid>
pct clone <ctid> <newid> --hostname <name>
pct destroy <ctid>

# Bind/mount a host path into the container
pct set <ctid> -mp0 /host/path,mp=/container/path

# Move files in/out
pct push <ctid> <local-file> <container-path>
pct pull <ctid> <container-path> <local-file>
```

> [!key-insight]
> `--unprivileged 1` should be the default; add `--features nesting=1` only when
> the workload needs it (systemd, Docker, or another LXC inside). Privileged
> containers share the host's root namespace — reserve them for the rare case
> that genuinely needs it.

## VM Management

```bash
qm list                                # all VMs on this node
qm start/stop/shutdown/reboot <vmid>
qm clone <vmid> <newid> --full
qm migrate <vmid> <target-node> --online
qm set <vmid> --memory 4096 --cores 4

# Disk ops
qm importdisk <vmid> <image> <storage>
qm disk move   <vmid> scsi0 <new-storage>
qm disk resize <vmid> scsi0 +20G

# Guest-agent helpers (needs qemu-guest-agent in the VM)
qm agent <vmid> ping
qm guest exec <vmid> -- <command>
```

## ZFS on Proxmox

Full ZFS reference (pools, datasets, snapshots, `send`/`receive`, scrub,
sanoid/syncoid) → [[ZFS]]. The one knob you almost always set on a hypervisor is
the **ARC cap**, so the cache doesn't starve guest RAM (default is up to 50% of
system memory):

```bash
# /etc/modprobe.d/zfs.conf  — example: 8 GiB ARC cap
options zfs zfs_arc_max=8589934592
update-initramfs -u && reboot
```

Inspect and tune at runtime (lowering the cap takes effect without a reboot):

```bash
arc_summary | less                         # full ARC report (hit ratio, size, breakdown)
arcstat 1                                   # live hits/misses/ARC size, 1s interval
grep -E '^(size|c_max|c_min)' /proc/spl/kstat/zfs/arcstats   # current vs caps (bytes)
echo 8589934592 | tee /sys/module/zfs/parameters/zfs_arc_max # apply a new cap now
```

> [!key-insight]
> On a hypervisor the default ARC competes with guest memory. Pin `zfs_arc_max`
> to a deliberate ceiling (rule of thumb: ~1–2 GiB per 16 GiB of host RAM for a
> VM-heavy node) so the ARC is a cache, not a memory hog. Watch for the host
> looking "full" — that's usually ARC, and `arc_summary` confirms it. VM disks
> live on zvols — Proxmox's ZFS storage plugin manages those for you; set
> `volblocksize` on the storage, not `recordsize`. See [[ZFS]] for ARC/L2ARC
> tuning and the rest.

## Proxmox Backup Server (PBS)

PBS gives **incremental, deduplicated, verifiable** backups — far better than
plain `vzdump` dumps. After the first full backup, the QEMU **dirty-bitmap**
means subsequent runs only ship changed blocks.

```bash
# On PVE: add the PBS datastore as storage (or via GUI: Datacenter → Storage)
# Then back up a guest to it:
vzdump <vmid> --storage <pbs-storage> --mode snapshot

# From a client host, talk to PBS directly:
export PBS_REPOSITORY="<user>@pbs@<pbs-host>:<datastore>"
proxmox-backup-client backup root.pxar:/ --ns <namespace>
proxmox-backup-client snapshot list
proxmox-backup-client restore <snapshot> root.pxar /restore/target
```

PBS-side hygiene (GUI or CLI/API): **prune** by retention, **garbage-collect**
to reclaim unreferenced chunks, and **verify** to catch bit-rot:

```bash
proxmox-backup-manager prune-job  ...      # or set a schedule in the GUI
proxmox-backup-manager garbage-collection start <datastore>
proxmox-backup-manager verify <datastore>
```

> [!key-insight]
> Dedup is global per datastore, so 20 near-identical VMs cost roughly one VM of
> space. But dedup also means a corrupt chunk affects every snapshot that
> references it — schedule **verify** jobs so rot is caught early, not at restore
> time.

## Notes

- For TLS certificates fronting the web UI, see [[Let's Encrypt - Certbot|Let's Encrypt / Certbot]].
- macOS guests require a special board key; see [[macOS Virtualization]].
- ZFS snapshots are not backups — pair them with PBS for the 3-2-1 picture ([[3-2-1 Backup Strategy]]).
