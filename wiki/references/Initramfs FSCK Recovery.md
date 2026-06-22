---
type: reference
title: "Initramfs FSCK Recovery"
created: 2026-06-21
updated: 2026-06-21
tags:
  - linux
  - recovery
  - filesystem
  - runbook
status: seed
related:
  - "[[Btrfs Restore From Snapshot]]"
---

# Initramfs FSCK Recovery

Runbook for a Linux system (commonly an LVM-root VM) that drops to an
`(initramfs)` prompt with filesystem errors after an unclean shutdown — power
loss, forced VM stop, host crash, or storage interruption.

Typical symptoms:

```text
/dev/mapper/<vg>-<lv>: UNEXPECTED INCONSISTENCY; RUN fsck MANUALLY
```

or

```text
The root filesystem on /dev/mapper/<vg>-<lv> requires a manual fsck
```

## 1. Identify the root device

At the `(initramfs)` prompt:

```bash
ls /dev/mapper
# e.g. control  <vg>-<lv>
```

> [!key-insight]
> LVM device names use **double hyphens** to escape literal hyphens in the
> VG/LV names. `/dev/mapper/ubuntu--vg-ubuntu--lv` is one device — never add
> spaces around the hyphens.

## 2. Run the filesystem repair

```bash
fsck.ext4 -f -y /dev/mapper/<vg>-<lv>
```

| Flag | Meaning |
|------|---------|
| `-f` | Force a check even if the FS looks clean |
| `-y` | Auto-answer "yes" to every repair prompt |

Expect passes 1–5 and possibly:

```text
***** FILE SYSTEM WAS MODIFIED *****
```

— which means corruption was repaired.

## 3. Handle I/O errors

If you see `I/O error` / `Buffer I/O error` / `Error reading block` and fsck
prompts `Ignore error?` or `Force rewrite?`, answer `yes` to continue. If fsck
completes, the filesystem is usually recoverable — but repeated I/O errors hint at
**underlying storage problems**, not just FS corruption.

## 4. Reboot

```bash
reboot -f       # or: exit  (to continue booting)
```

## 5. Post-recovery validation

```bash
sudo dmesg -T | grep -Ei "error|fail|ext4"
df -h
mount | grep '^/dev'
journalctl -b -p warning
```

## 6. If virtualized — check the host

I/O errors during fsck warrant inspecting the hypervisor's storage:

```bash
dmesg -T | grep -Ei "error|fail|i/o"
journalctl -p err -b
zpool status -v        # ZFS-backed storage
pvs; vgs; lvs -a       # LVM / LVM-thin
```

## Notes

- FS corruption after a dirty shutdown is common and usually recoverable.
- **Always have current backups before major FS repairs** — see
  [[Btrfs Restore From Snapshot]] for the snapshot-based recovery path on btrfs
  roots.

## Related

- [[Btrfs Restore From Snapshot]] — equivalent recovery on a btrfs root
