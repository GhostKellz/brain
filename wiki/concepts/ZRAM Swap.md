---
type: concept
title: "ZRAM Swap"
created: 2026-06-21
updated: 2026-06-21
tags:
  - linux
  - zram
  - memory
  - swap
status: seed
related:
  - "[[Linux Memory Tuning]]"
  - "[[Sysctl Performance Tuning]]"
---

# ZRAM Swap

**ZRAM** is a compressed block device that lives in RAM. Used as swap, it lets the
kernel "swap out" pages by **compressing them in memory** instead of writing to
disk — far faster than disk swap, at the cost of some CPU and the RAM the
compressed data occupies.

> [!key-insight]
> ZRAM trades CPU for effective memory: a page that compresses 3:1 effectively
> frees two-thirds of its footprint. But the compressed data still lives in RAM,
> so ZRAM is not "free extra memory" — it's "more usable memory if your data is
> compressible."

## Configuration (systemd-zram-generator)

```ini
# /etc/systemd/zram-generator.conf
[zram0]
zram-size = 16384              # MiB; a fixed size, or e.g. "ram / 2"
compression-algorithm = zstd   # good ratio/speed balance
```

Apply by rebooting (the generator runs at boot). Check status:

```bash
swapon --show --bytes
cat /etc/systemd/zram-generator.conf
```

## Sizing — bounded beats huge

On RAM-rich machines, `ram / 2` can be counterproductive: a large ZRAM lets the
system thrash through gigabytes of compressed pressure before the OOM killer
fires, causing freezes. A **bounded fixed size** (e.g. 16 GB) gives swap headroom
while ensuring faster OOM intervention. This is the "fail fast" reasoning of
[[Linux Memory Tuning]].

## Compression algorithm

| Algorithm | Ratio | Speed |
|-----------|-------|-------|
| `lzo` / `lzo-rle` | lower | fastest |
| `zstd` | higher | balanced (common default) |
| `lz4` | medium | very fast |

`zstd` is the usual pick for a desktop — strong ratio without much CPU cost.

## ZRAM vs zswap

ZRAM *is* swap (compressed, in RAM). zswap is a compressed cache in front of
*disk* swap. Use one or the other; ZRAM-on / zswap-off is the common combo. See
[[Linux Memory Tuning]].

## Related

- [[Linux Memory Tuning]] — the broader "fail fast" philosophy
- [[Sysctl Performance Tuning]] — swappiness and dirty ratios that interact with swap
