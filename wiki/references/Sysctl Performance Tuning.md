---
type: reference
title: "Sysctl Performance Tuning"
created: 2026-06-21
updated: 2026-06-21
tags:
  - sysctl
  - kernel
  - performance
  - linux
status: seed
related:
  - "[[Linux Memory Tuning]]"
  - "[[nftables Firewall]]"
  - "[[Disk IO Scheduler]]"
---

# Sysctl Performance Tuning

`sysctl` exposes kernel runtime parameters. On a systemd system, files in
`/etc/sysctl.d/*.conf` are applied at boot by `systemd-sysctl` (numeric prefixes
order them). Apply without rebooting via `sudo sysctl --system`.

```bash
sudo sysctl --system          # reload all *.conf
sysctl vm.swappiness          # read one value
```

## Memory / VM tunables

```ini
# /etc/sysctl.d/99-sysctl.conf
vm.swappiness = 10            # bias away from swapping out anon pages
vm.vfs_cache_pressure = 50    # keep inode/dentry caches longer
vm.dirty_ratio = 15           # cap on dirty pages before forced writeback
vm.dirty_background_ratio = 5 # threshold to start background writeback
```

- **`vm.swappiness`** — lower keeps app pages in RAM longer (good on RAM-rich
  desktops). See [[Linux Memory Tuning]].
- **`vm.vfs_cache_pressure`** — lower preserves filesystem metadata caches,
  improving FS-heavy workloads.
- **dirty ratios** — lower forces earlier disk writeback, avoiding big dirty-page
  buildups that cause I/O stalls under heavy memory activity.

## Gaming compatibility

```ini
# /etc/sysctl.d/80-gamecompatibility.conf
vm.max_map_count = 2147483642
```

Many modern games under Proton/Wine/Lutris — especially Unreal/Unity titles and
anti-cheat (EAC/BattlEye) — require a very high `max_map_count` or they crash.

## Network / conntrack (firewall hosts)

For high-throughput routers/firewalls running [[nftables Firewall]]:

```ini
net.netfilter.nf_conntrack_max = 262144
net.netfilter.nf_conntrack_buckets = 65536
net.netfilter.nf_conntrack_tcp_timeout_established = 3600
net.netfilter.nf_conntrack_tcp_timeout_time_wait = 30
```

## Verifying

```bash
sysctl vm.swappiness vm.vfs_cache_pressure vm.max_map_count
cat /etc/sysctl.d/*            # what's configured
```

> [!key-insight]
> Prefer dropping files in `/etc/sysctl.d/` over editing `/etc/sysctl.conf` — it's
> modular, reload-friendly, and survives package updates cleanly.

## Related

- [[Linux Memory Tuning]] — ZRAM, OOM, swap philosophy
- [[Disk IO Scheduler]] — the I/O side of responsiveness
- [[nftables Firewall]] — conntrack sizing context
