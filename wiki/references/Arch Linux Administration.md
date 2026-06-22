---
type: reference
title: "Arch Linux Administration"
created: 2026-06-21
updated: 2026-06-21
tags:
  - arch
  - pacman
  - aur
  - btrfs
status: developing
related:
  - "[[Linux Administration]]"
---

> [!key-insight] Generalized from field notes; host/client-specific values are placeholders.

Arch-specific package management, AUR, snapshots, and recovery.

## pacman Basics

```bash
sudo pacman -Syu          # full system upgrade (sync + update)
sudo pacman -Syyu         # force refresh all package DBs, then upgrade
sudo pacman -S <pkg>      # install
sudo pacman -Rns <pkg>    # remove with deps + config
```

### Tuning `/etc/pacman.conf`

Uncomment `Color` and `ParallelDownloads`; optionally add the `ILoveCandy` easter egg under `[options]`.

### Refresh mirrors with reflector

```bash
sudo pacman -S reflector
sudo cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.bak
sudo reflector --verbose --latest 10 --protocol https --sort rate --save /etc/pacman.d/mirrorlist
```

If using the Chaotic-AUR and hitting 404/connection errors:

```bash
sudo pacman -Sy chaotic-mirrorlist
sudo pacman -Syyu
```

## AUR (via `yay`)

```bash
sudo pacman -S --needed base-devel git
git clone https://aur.archlinux.org/yay.git
cd yay && makepkg -si

yay -S <pkg>      # install from AUR
yay -Rns <pkg>    # remove
yay -Syu          # upgrade everything
```

## Manual makepkg

```bash
git clone https://aur.archlinux.org/<pkg>.git
cd <pkg>
makepkg -si                 # build + install
makepkg -si --skipinteg     # skip checksum verification (use with care)
updpkgsums                  # update b2/sha sums in PKGBUILD
```

## Btrfs Snapshots with Snapper

```bash
sudo pacman -S snapper snap-pac
sudo btrfs subvolume create /.snapshots
sudo chmod 750 /.snapshots
sudo chown :wheel /.snapshots

# Add an fstab entry for the .snapshots subvol, e.g.:
# UUID=<root-uuid>  /.snapshots  btrfs  subvol=@.snapshots,noatime,compress=zstd:3  0 0
sudo mount /.snapshots

sudo snapper -c root create --description "Initial root snapshot"
sudo snapper -c root list

sudo snapper -c home create-config /home
sudo chmod 750 /home/.snapshots
sudo chown :wheel /home/.snapshots
sudo snapper -c home create --description "initial home snapshot"

sudo systemctl enable --now snapper-timeline.timer
sudo systemctl enable --now snapper-cleanup.timer
```

## Recovery / Misc

```bash
sudo mkinitcpio -P                          # rebuild all initramfs images
journalctl -b0 --grep="nvidia" --no-pager   # check current-boot NVIDIA logs
```

## Notes

- General cross-distro tasks: [[Linux Administration]].
