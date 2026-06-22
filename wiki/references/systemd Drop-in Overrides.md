---
type: reference
title: "systemd Drop-in Overrides"
created: 2026-06-21
updated: 2026-06-21
tags:
  - systemd
  - linux
  - configuration
status: seed
related:
  - "[[systemd Timers]]"
  - "[[Ollama Service Configuration]]"
---

# systemd Drop-in Overrides

A **drop-in** lets you customize a packaged systemd unit *without editing the
unit file itself*. You add a small `.conf` snippet in a `<unit>.d/` directory;
systemd merges it on top of the shipped unit. The packaged file stays pristine,
so **package updates can't clobber your changes** and **your changes can't be lost
in an update**.

> [!key-insight]
> Never edit `/usr/lib/systemd/system/foo.service` directly — a package upgrade
> overwrites it. Use a drop-in in `/etc/systemd/system/foo.service.d/` instead.

## Creating one

The guided way (opens an editor, creates the dir + file for you):

```bash
sudo systemctl edit foo.service
```

The manual way:

```
# /etc/systemd/system/foo.service.d/override.conf
[Service]
Environment="SOME_VAR=value"
```

Then reload + restart:

```bash
sudo systemctl daemon-reload
sudo systemctl restart foo.service
```

## How merging works

- Most directives in a drop-in **add to / override** the base unit's value.
- **List-valued** directives like `ExecStart=` and `Environment=` are special: to
  *replace* them you must first **clear** them with an empty assignment.

```ini
[Service]
ExecStart=                 # clear the inherited ExecStart first
ExecStart=/new/command     # then set the new one
```

## The wholesale-overwrite footgun

> [!key-insight]
> A drop-in is still one file. If you regenerate it with a blunt
> `tee > override.conf` that writes only one setting, **every other line in that
> drop-in disappears**. Always read the existing drop-in, edit in place, then
> verify the whole resulting environment — e.g.
> `systemctl show foo -p Environment`. This is exactly how an Ollama config once
> silently lost its models dir; see [[Ollama Service Configuration]].

## Verifying what's actually applied

```bash
systemctl cat foo.service              # base unit + all drop-ins, in merge order
systemctl show foo.service -p Environment
sudo systemd-delta                     # see all overridden/extended units system-wide
```

## Related

- [[systemd Timers]] — often customized via drop-ins
- [[Ollama Service Configuration]] — a real-world drop-in (and its footgun)
