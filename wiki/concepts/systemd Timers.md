---
type: concept
title: "systemd Timers"
created: 2026-06-21
updated: 2026-06-21
tags:
  - systemd
  - linux
  - automation
status: seed
related:
  - "[[systemd]]"
  - "[[Restic Backup]]"
  - "[[Snapper]]"
---

# systemd Timers

A **systemd timer** is a unit that activates another unit (usually a `.service`)
on a schedule — the systemd-native replacement for cron. A timer always pairs with
a service of the same name: `foo.timer` triggers `foo.service`.

> [!key-insight]
> Timers beat cron for system tasks because they integrate with the journal
> (`journalctl -u foo`), respect dependencies (`After=network-online.target`),
> can catch up missed runs (`Persistent=true`), and run jobs in proper cgroups.

## The two-file pattern

`foo.service` (a oneshot — does the work):

```ini
[Unit]
Description=Do the thing
Wants=network-online.target
After=network-online.target

[Service]
Type=oneshot
ExecStart=/usr/bin/do-the-thing
```

`foo.timer` (when to run it):

```ini
[Unit]
Description=Run do-the-thing daily

[Timer]
OnCalendar=daily        # or OnCalendar=*-*-* 03:00:00
Persistent=true         # run on next boot if a scheduled run was missed
OnBootSec=15min         # alternatively: relative to boot

[Install]
WantedBy=timers.target
```

Enable the **timer**, not the service:

```bash
sudo systemctl enable --now foo.timer
```

## OnCalendar vs OnBootSec/OnUnitActiveSec

- **`OnCalendar=`** — wall-clock schedules (`daily`, `weekly`,
  `*-*-* 03:00:00`). Validate expressions with `systemd-analyze calendar`.
- **`OnBootSec=` / `OnUnitActiveSec=`** — relative timers (X after boot, or X
  after the unit last ran).

## Inspecting

```bash
systemctl list-timers              # all timers, next/last fire times
systemctl status foo.timer
journalctl -u foo.service          # output from the triggered service
```

## Where it shows up here

- [[Restic Backup]] — a daily backup `.service` + `.timer`.
- [[Snapper]] — `snapper-timeline.timer` and `snapper-cleanup.timer`.
- Any periodic maintenance (weekly update scripts, etc.).

> [!key-insight]
> `Persistent=true` is what makes timers right for desktops/laptops that aren't
> always on — a missed daily job fires at next boot instead of being skipped.

## Related

- [[systemd]] — customizing units without editing them (drop-ins)
- [[Restic Backup]] / [[Snapper]] — concrete timer-driven jobs
