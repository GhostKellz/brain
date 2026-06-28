---
type: reference
title: systemd
aliases:
  - systemd Drop-in Overrides
  - references/systemd-Drop-in-Overrides
created: 2026-06-21
updated: 2026-06-28
tags:
  - systemd
  - linux
  - services
  - journald
  - cgroups
  - sysadmin
status: developing
related:
  - "[[systemd Timers]]"
  - "[[Ollama Service Configuration]]"
  - "[[Linux Administration]]"
  - "[[Sysctl Performance Tuning]]"
---

> [!key-insight] systemd is the common layer across every distro in this vault. Master units, drop-ins, journald, and the resource-control/hardening directives and most "how do I make this run reliably" questions answer themselves.

The init system + service manager. This is the leverage note: unit model,
`systemctl`/`journalctl` fluency, drop-in overrides, and the parts people skip —
cgroup resource control, service sandboxing, and boot analysis.

Timers get their own page: [[systemd Timers]].

## Unit model

systemd manages **units** — typed objects with a `<name>.<type>` suffix:

| Type | What it manages |
|------|-----------------|
| `.service` | a process/daemon |
| `.socket` | socket activation (start a service on first connection) |
| `.timer` | scheduled activation (cron replacement → [[systemd Timers]]) |
| `.target` | a sync point / group of units (runlevel-like) |
| `.mount` / `.automount` | filesystem mounts (generated from fstab) |
| `.path` | activate a unit when a path changes |
| `.slice` | a cgroup branch for resource control |

Unit file search order (later wins, drop-ins merge on top):

```
/usr/lib/systemd/system/   # packaged units — never edit
/etc/systemd/system/       # admin units + drop-ins — your edits live here
```

## systemctl essentials

```bash
sudo systemctl start|stop|restart|reload foo.service
sudo systemctl enable --now foo        # enable at boot + start now
sudo systemctl disable --now foo       # disable + stop
sudo systemctl mask foo                # make it impossible to start (symlink to /dev/null)
sudo systemctl unmask foo

systemctl status foo                   # state + recent logs + cgroup tree
systemctl is-active foo                # active | inactive | failed
systemctl is-enabled foo               # enabled | disabled | masked
systemctl is-failed foo
sudo systemctl daemon-reload           # re-read unit files after editing
```

## Inspecting & discovery

```bash
systemctl cat foo                      # base unit + all drop-ins, in merge order
systemctl show foo                     # every resolved property
systemctl show foo -p Environment      # one property
systemctl list-units --type=service    # active units
systemctl list-units --state=failed    # what's broken
systemctl list-unit-files --state=enabled
systemctl list-dependencies foo        # dependency tree
systemctl list-dependencies foo --reverse   # what depends on foo
```

## Drop-in overrides

A **drop-in** customizes a packaged unit *without editing the unit file itself*.
You add a small `.conf` snippet in a `<unit>.d/` directory; systemd merges it on
top. The packaged file stays pristine, so **package updates can't clobber your
changes** and **your changes can't be lost in an update**.

> [!key-insight]
> Never edit `/usr/lib/systemd/system/foo.service` directly — a package upgrade
> overwrites it. Use a drop-in in `/etc/systemd/system/foo.service.d/` instead.

The guided way (creates the dir + file for you):

```bash
sudo systemctl edit foo.service        # override snippet only
sudo systemctl edit --full foo.service # fork the whole unit into /etc (rarely needed)
```

The manual way:

```ini
# /etc/systemd/system/foo.service.d/override.conf
[Service]
Environment="SOME_VAR=value"
```

```bash
sudo systemctl daemon-reload
sudo systemctl restart foo.service
```

### How merging works

- Most directives in a drop-in **add to / override** the base unit's value.
- **List-valued** directives like `ExecStart=` and `Environment=` are special: to
  *replace* them you must first **clear** them with an empty assignment.

```ini
[Service]
ExecStart=                 # clear the inherited ExecStart first
ExecStart=/new/command     # then set the new one
```

> [!warning] The wholesale-overwrite footgun
> A drop-in is still one file. If you regenerate it with a blunt
> `tee > override.conf` that writes only one setting, **every other line in that
> drop-in disappears**. Always read the existing drop-in, edit in place, then
> verify the whole resulting state — e.g. `systemctl show foo -p Environment`.
> This is exactly how an Ollama config once silently lost its models dir; see
> [[Ollama Service Configuration]].

### Verifying what's applied

```bash
systemctl cat foo.service              # base unit + all drop-ins
systemctl show foo.service -p Environment
sudo systemd-delta                     # all overridden/extended units system-wide
```

## Logs (journald)

```bash
journalctl -u foo                      # one unit
journalctl -u foo -f                   # follow (tail -f)
journalctl -u foo --since "1 hour ago"
journalctl -b                          # this boot
journalctl -b -1                       # previous boot
journalctl -p err -b                   # errors this boot (emerg..debug)
journalctl -k                          # kernel ring buffer (dmesg)
journalctl --disk-usage
sudo journalctl --vacuum-time=2weeks   # trim retention
sudo journalctl --vacuum-size=500M
```

Persistent logs: ensure `Storage=persistent` in `/etc/systemd/journald.conf` and
`/var/log/journal/` exists.

## Targets & boot

```bash
systemctl get-default                  # usually graphical.target or multi-user.target
sudo systemctl set-default multi-user.target   # headless servers: no GUI
sudo systemctl isolate rescue.target   # drop to single-user-ish (maintenance)
systemctl list-units --type=target
```

## Resource control (cgroups)

Every service lives in a cgroup; cap its resources with drop-in directives —
no external cgroup tooling needed.

```ini
# /etc/systemd/system/foo.service.d/limits.conf
[Service]
CPUQuota=50%            # max half of one CPU
MemoryMax=512M          # hard cap — OOM-kills the unit if exceeded
MemoryHigh=400M         # soft throttle before the hard cap
TasksMax=100            # PID/thread limit
IOWeight=50             # relative disk-IO priority (1..10000)
```

```bash
systemctl status foo                   # shows live Memory/CPU/Tasks
systemd-cgtop                          # top-like view of cgroup resource use
systemctl show foo -p MemoryMax -p CPUQuotaPerSecUSec
```

Group related services under a custom **slice** to cap them collectively
(`Slice=mygroup.slice` in the unit + a `mygroup.slice` defining the totals).

## Service hardening / sandboxing

systemd can sandbox a daemon with one-line directives — cheap defense-in-depth.

```ini
[Service]
NoNewPrivileges=true       # block setuid privilege escalation
ProtectSystem=strict       # mount /usr, /boot, /etc read-only
ProtectHome=true           # hide /home, /root, /run/user
PrivateTmp=true            # private /tmp namespace
PrivateDevices=true        # minimal /dev, no raw device access
ProtectKernelTunables=true # /proc/sys, /sys read-only
ProtectKernelModules=true  # block module load/unload
RestrictAddressFamilies=AF_INET AF_INET6   # allowed socket families
SystemCallFilter=@system-service           # seccomp allowlist
CapabilityBoundingSet=CAP_NET_BIND_SERVICE # drop all caps except this
ReadWritePaths=/var/lib/foo                # carve out writable dirs under ProtectSystem
```

> [!key-insight]
> Add these via a drop-in on a packaged unit, then audit with
> `systemd-analyze security foo` — it scores the unit 0-10 (lower = safer) and
> lists exactly which directives would tighten it.

## systemd-analyze

```bash
systemd-analyze                        # total boot time
systemd-analyze blame                  # slowest units at boot
systemd-analyze critical-chain         # the boot critical path
systemd-analyze security               # exposure score for ALL services
systemd-analyze security foo           # detailed hardening report for one unit
systemd-analyze verify foo.service     # lint a unit file for errors
```

## User services

Per-user units, no root needed (`~/.config/systemd/user/`):

```bash
systemctl --user enable --now foo
systemctl --user status foo
journalctl --user -u foo
loginctl enable-linger <user>          # let user services run without an active login
```

## Related

- [[systemd Timers]] — scheduled activation (cron replacement)
- [[Ollama Service Configuration]] — a real-world drop-in (and its footgun)
- [[Linux Administration]] — the cross-distro hub
- [[Sysctl Performance Tuning]] — kernel tunables (complementary to cgroup limits)
