---
type: entity
title: "Restic"
created: 2026-06-21
updated: 2026-06-21
tags:
  - restic
  - backup
  - encryption
status: seed
related:
  - "[[Restic Backup]]"
  - "[[3-2-1 Backup Strategy]]"
---

# Restic

**Restic** is a fast, cross-platform backup program that produces **encrypted,
deduplicated** snapshots into a repository. The repo can live on local disk,
SFTP, or S3-compatible object storage (AWS S3, MinIO, Wasabi, Backblaze B2).

## Why it's a good default

- **Encrypted by design** — the repo is useless without the password; safe on
  untrusted media.
- **Content-addressed dedup** — only changed chunks are stored, so daily backups
  are cheap.
- **Single static binary** — no daemon, trivial to script and run from a
  [[systemd Timers|systemd timer]].
- **Snapshot model** — each backup is a point-in-time snapshot you can `restore`,
  `mount` (FUSE), or `diff`.

## Core verbs

```bash
restic init                 # create a repo
restic backup <paths>       # take a snapshot
restic snapshots            # list snapshots
restic restore <id> --target <dir>
restic mount <dir>          # browse snapshots as a filesystem
restic forget --prune ...   # apply retention + reclaim space
restic check                # verify repository integrity
```

## Configuration model

Repository location and credentials are supplied via environment variables
(`RESTIC_REPOSITORY`, `RESTIC_PASSWORD`, and the cloud provider's keys), which is
why an `EnvironmentFile` pattern is the clean way to run it under systemd. Full
setup in [[Restic Backup]].

## Where it fits

Restic is the **encrypted, file-level, off-disk** tier of a
[[3-2-1 Backup Strategy]] — complementary to [[Btrfs Snapshots]] (instant local
rollback) rather than a replacement.

## Related

- [[Restic Backup]] — setup + systemd automation
- [[3-2-1 Backup Strategy]] — the broader strategy
