---
type: reference
title: "Restic Backup"
created: 2026-06-21
updated: 2026-06-21
tags:
  - restic
  - backup
  - systemd
  - encryption
status: seed
related:
  - "[[Restic]]"
  - "[[3-2-1 Backup Strategy]]"
  - "[[systemd Timers]]"
---

# Restic Backup

How to set up [[Restic]] for encrypted, deduplicated, automated backups driven by
a [[systemd Timers|systemd timer]]. This is the file-backup tier of a
[[3-2-1 Backup Strategy]].

## Install + initialise

```bash
sudo pacman -S restic
```

Store repo + credentials in an environment file so they never live in the
service or scripts (S3/MinIO example):

```ini
# /etc/restic.env  (chmod 640, root-owned)
RESTIC_REPOSITORY="s3:http://<s3-endpoint>/<bucket>"
AWS_ACCESS_KEY_ID="<access-key>"
AWS_SECRET_ACCESS_KEY="<secret-key>"
RESTIC_PASSWORD="<encryption-password>"
```

> [!gap]
> Endpoints, bucket names, and keys are secrets/infra — placeholders only here.
> The real values live in the private tier.

Initialise the repo once:

```bash
source /etc/restic.env
restic init
restic backup /home          # first manual run
restic snapshots             # verify
```

## Retention / pruning

```bash
restic forget --keep-daily 7 --keep-weekly 4 --keep-monthly 6 --prune
```

This keeps a useful history without unbounded growth.

## Automate with systemd

`restic-backup.service` (oneshot — backup then prune):

```ini
[Unit]
Description=Restic backup
Wants=network-online.target
After=network-online.target

[Service]
Type=oneshot
EnvironmentFile=/etc/restic.env
ExecStart=/usr/bin/restic backup /home
ExecStartPost=/usr/bin/restic forget --keep-daily 7 --keep-weekly 4 --keep-monthly 6 --prune
Nice=19
IOSchedulingClass=2
IOSchedulingPriority=7
StandardOutput=append:/var/log/restic.log
StandardError=append:/var/log/restic.log
```

`restic-backup.timer`:

```ini
[Unit]
Description=Daily restic backup

[Timer]
OnCalendar=daily
Persistent=true

[Install]
WantedBy=timers.target
```

Enable + test:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now restic-backup.timer
sudo systemctl start restic-backup.service   # run now
sudo tail -f /var/log/restic.log
```

> [!key-insight]
> `Persistent=true` makes the timer fire on next boot if the machine was off when
> it was scheduled — important for desktops that aren't always on.

## Why this shape

- **Encrypted at rest** by restic natively — safe to store on a NAS or in a
  bucket you don't fully trust.
- **Deduplicated** — repeated backups are cheap.
- **Credentials out of the unit** via `EnvironmentFile`.
- **Low priority I/O** (`Nice` + `IOSchedulingPriority`) so backups don't fight
  interactive work.

## Related

- [[Restic]] — the tool itself
- [[3-2-1 Backup Strategy]] — where this tier fits
- [[systemd Timers]] — scheduling mechanism
