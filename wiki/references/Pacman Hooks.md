---
type: reference
title: "Pacman Hooks"
created: 2026-06-21
updated: 2026-06-21
tags:
  - pacman
  - arch-linux
  - automation
status: seed
related:
  - "[[NVIDIA Container Runtime Troubleshooting]]"
  - "[[PKGBUILD Templates]]"
---

# Pacman Hooks

A **pacman hook** runs a command automatically before or after a package
transaction, triggered by changes to specific packages or files. They live in
`/etc/pacman.d/hooks/*.hook` (user/admin hooks) and let you automate
post-install steps that would otherwise be easy to forget.

> [!key-insight]
> Hooks turn "remember to run X after upgrading Y" into "the system runs X for
> you." The canonical example: regenerate the NVIDIA CDI spec whenever the driver
> upgrades, so GPU containers never break silently. See
> [[NVIDIA Container Runtime Troubleshooting]].

## Hook anatomy

```ini
# /etc/pacman.d/hooks/example.hook
[Trigger]
Operation = Install
Operation = Upgrade
Operation = Remove          # any subset
Type = Package              # or: Path (watch files in the package)
Target = some-package       # globs allowed; multiple Target lines OR together

[Action]
Description = Doing the thing after a transaction...
When = PostTransaction      # or PreTransaction
Exec = /usr/bin/do-the-thing
NeedsTargets                # pass matched targets to Exec on stdin
Depends = some-tool         # ensure a tool is installed before running
```

## Trigger types

- **`Type = Package`** — fire when named packages are installed/upgraded/removed.
- **`Type = Path`** — fire when files matching a path glob change (e.g.
  `usr/lib/modules/*` to rebuild things on kernel updates).

## Timing

- **`PreTransaction`** — before pacman makes changes (e.g. snapshot first).
- **`PostTransaction`** — after (e.g. rebuild caches, regenerate config).

## Worked example — auto-fix NVIDIA CDI

```ini
# /etc/pacman.d/hooks/nvidia-container-toolkit.hook
[Trigger]
Operation = Install
Operation = Upgrade
Type = Package
Target = nvidia-utils
Target = nvidia-open-dkms

[Action]
Description = Regenerating NVIDIA CDI spec for container toolkit...
When = PostTransaction
Exec = /usr/bin/nvidia-ctk cdi generate --output=/etc/cdi/nvidia.yaml
NeedsTargets
```

## Where hooks come from

- **System hooks** ship in `/usr/share/libalpm/hooks/` (don't edit these).
- **Your hooks** go in `/etc/pacman.d/hooks/`. A file there with the same name as
  a system hook **overrides** it — a way to disable or replace stock behaviour.

## Related

- [[NVIDIA Container Runtime Troubleshooting]] — the motivating use case
- [[PKGBUILD Templates]] — `.install` scripts are a related per-package hook mechanism
