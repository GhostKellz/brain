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

> [!key-insight]
> On a hypervisor the default ARC competes with guest memory. Pin `zfs_arc_max`
> to a deliberate ceiling (rule of thumb: ~1–2 GiB per 16 GiB of host RAM for a
> VM-heavy node) so the ARC is a cache, not a memory hog. VM disks live on
> zvols — Proxmox's ZFS storage plugin manages those for you; set
> `volblocksize` on the storage, not `recordsize`. See [[ZFS]] for the rest.

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
