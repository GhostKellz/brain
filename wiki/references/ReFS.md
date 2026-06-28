---
type: reference
title: "ReFS"
created: 2026-06-28
updated: 2026-06-28
aliases:
  - "Resilient File System"
  - "Block Cloning"
  - "Integrity Streams"
tags:
  - refs
  - windows
  - filesystem
  - backup
  - storage
status: developing
related:
  - "[[Veeam]]"
  - "[[Hyper-V Administration]]"
  - "[[Windows Administration]]"
  - "[[Btrfs]]"
  - "[[ZFS]]"
---

# ReFS

**ReFS (Resilient File System)** is Microsoft's modern, copy-on-write filesystem
built for data integrity, huge scale, and — the reason it matters here — **block
cloning**, which makes it the preferred filesystem for a [[Veeam]] backup
repository. It's the Windows-world cousin of [[Btrfs]]/[[ZFS]]: checksummed,
CoW, and self-healing on redundant storage.

> [!key-insight]
> NTFS overwrites in place and trusts the disk; ReFS is **copy-on-write** and
> **checksums everything**, so it can *detect* bit-rot and (on a mirror/parity
> Storage Space) *auto-repair* it from the good copy. For backups the killer
> feature is **block cloning**: copying a range of blocks becomes a metadata
> pointer update instead of a physical read+write — which is exactly what turns a
> Veeam synthetic full from an hours-long rewrite into a near-instant, near-zero-
> space operation.

## ReFS vs NTFS

| | NTFS | ReFS |
|--|------|------|
| Write model | In-place | **Copy-on-write** (allocate-on-write) |
| Integrity | None on data | **Checksums** on metadata (+ optional data) |
| Self-heal | No | Yes, with Storage Spaces mirror/parity |
| Block cloning | **No** | **Yes** (reflink-style) |
| Max volume | 256 TB | **35 PB** |
| Compatibility | Universal (boot, removable, all tools) | Server/data volumes; not a boot disk |
| SAN features | thin provisioning, TRIM/UNMAP, ODX | Limited (use NTFS if you need ODX) |

The trade is **resilience + scale** (ReFS) vs **universal compatibility** (NTFS).
ReFS can't be a boot volume and lacks some SAN offload features, so it's a
*data/backup-repository* filesystem, not a general replacement.

## How the key features work

### Block cloning (the Veeam lever)

Block cloning lets the filesystem copy a byte range as a **low-cost metadata
operation**: multiple files share the same logical clusters via **reference
counting**, so a "copy" just remaps regions instead of moving data. When a shared
region is later written, ReFS does **allocate-on-write** — a new block for the
change — keeping files isolated. This is what Veeam's **Fast Clone** rides on to
build synthetic fulls and merge/transform backup chains without rewriting blocks
(up to ~4× faster merges, minimal extra space).

### Integrity streams + scrubber

ReFS keeps checksums and validates on read; a background **scrubber** periodically
scans the volume to catch latent corruption and repair it (with a redundant
Storage Space).

> [!warning]
> **Integrity streams are usually disabled on backup/VHDX data.** Checksum-on-write
> for large, constantly-rewritten VHDX files causes a "write amplification" loop
> that hurts performance and stability. Backup vendors (Veeam) and FSLogix/profile
> guidance commonly disable integrity streams on the data files. Note: for **block
> cloning** to work, source and destination must have the **same** integrity-stream
> setting.

### Mirror-accelerated parity (MAP)

On **Storage Spaces Direct**, ReFS can blend resiliencies: writes land in a fast
**mirror** tier and cold data rotates (in 64 MiB regions) to space-efficient
**parity**. Microsoft recommends MAP **for backup/archival workloads only** — for
hot random VM workloads use three-way mirror. Parity destage can bottleneck, so
size the SSD/mirror tier generously.

## Using ReFS as a Veeam repository

> [!key-insight]
> **Format the repo volume ReFS with a 64 KB cluster size.** Backup I/O is large
> and sequential, Veeam blocks are >64 KB, and 64 KB clusters are where ReFS block
> cloning + sequential throughput shine (4 KB is Microsoft's general default but
> the wrong choice here). This is the single most important repo-formatting
> decision.

```powershell
# Format a data volume as ReFS with 64K clusters (PowerShell)
Format-Volume -DriveLetter E -FileSystem ReFS -AllocationUnitSize 65536 -NewFileSystemLabel "VeeamRepo"
```

- ReFS **3.1+** is required for Veeam Fast Clone; SMB targets need SMB **3.1.1**
  plus duplicate-extents support.
- The Linux equivalent is **XFS with `reflink=1`** — same Fast Clone benefit on a
  [[Linux Administration|Linux]] hardened repo. Pick ReFS on Windows, XFS on Linux.
- Windows Server **2025** matured ReFS further (e.g. Direct I/O for CSVs), narrowing
  the old "use NTFS for SANs" rule — but verify against current docs per backend.

## Pros / trade-offs

**Pros**
- Block cloning → fast, space-efficient Veeam synthetic fulls/merges.
- End-to-end checksums + scrubber + auto-heal (with Storage Spaces).
- Scales to petabytes; great for large backup repositories.

**Trade-offs**
- Not bootable; not for general-purpose/removable volumes.
- Integrity streams often must be **off** for VHDX/backup data (write amplification).
- Some SAN features (ODX, TRIM/UNMAP, thin provisioning) need NTFS.
- MAP requires Storage Spaces Direct; parity destage can be slow.

## Related

- [[Veeam]] — Fast Clone rides ReFS block cloning (format repos ReFS-64K)
- [[Hyper-V Administration]] — ReFS vs NTFS for VM storage / CSVs
- [[Windows Administration]] — host-side volume formatting/management
- [[Btrfs]] · [[ZFS]] — the Linux CoW/checksumming analogues (XFS reflink is the Fast-Clone peer)
