---
type: reference
title: "PKGBUILD Auditing"
created: 2026-06-21
updated: 2026-06-21
tags:
  - arch-linux
  - packaging
  - security
  - aur
status: seed
related:
  - "[[PKGBUILD Templates]]"
---

# PKGBUILD Auditing

A PKGBUILD is **arbitrary shell that runs as your user** (and `package()` /
`.install` hooks can run as root). Treat every one — yours or from the AUR — as
untrusted code until reviewed. This is a pre-build checklist.

## Tools

```sh
sudo pacman -S --needed namcap pacman-contrib devtools
```

| Tool | Use |
|------|-----|
| `namcap` | Lint the PKGBUILD and the built package |
| `makepkg --printsrcinfo` | Confirm metadata parses |
| `makepkg --verifysource` | Download + checksum sources *without building* |
| `updpkgsums` | Regenerate `sha256sums` |
| `extra-x86_64-build` (devtools) | Build in a throwaway clean chroot |

## Static review — read before you run

1. **Source provenance** — does `url`/`source` point at the real upstream? Watch
   for typo-squatted hosts or `source` host ≠ `url` host.
2. **`source` array** — only URLs, `vcs+url#fragment`, or local files. Reject
   anything that pipes to a shell (`curl ... | sh`), uses URL shorteners, or
   hardcodes a random IP.
3. **Checksums** — non-VCS sources need real `sha*sums`, not `SKIP`. `SKIP` is
   only legitimate for VCS sources (the commit hash is the integrity anchor) and
   PGP-verified signed sources (`validpgpkeys`).
4. **Functions** — read `prepare/build/check/package` line by line:
   - No network access outside `prepare()` (build/check should be offline).
   - No writes outside `$srcdir`/`$pkgdir`; nothing touching `$HOME`, `/etc`,
     `/usr` directly.
   - No `sudo`/privilege escalation inside any function.
   - Read any `*.install` hook — it runs as root.
5. **Dependencies** — real package names; no surprise toolchains or networking
   utilities that could exfiltrate.
6. **`options`** — disabled hardening (`!buildflags`, `!strip`, `!lto`) needs a
   reason.

## Mechanical checks

```sh
makepkg --printsrcinfo          # metadata sane?
makepkg --verifysource -f       # fetch + verify checksums, no build
namcap PKGBUILD                 # lint the recipe
makepkg -f && namcap ./*.pkg.tar.zst   # build, then lint the artifact
```

## Build in isolation

> [!key-insight]
> A **clean-chroot build** fails if any dependency isn't declared — the single
> most reliable correctness check for a PKGBUILD, and it keeps untrusted build
> steps off your live system.

```sh
extra-x86_64-build              # devtools: builds in a throwaway clean chroot
```

## Red flags — stop and investigate

- `sha256sums=('SKIP')` on a plain tarball/HTTP source.
- Base64/hex blobs decoded and executed in any function.
- `source` host differs from `url` host with no explanation.
- Build steps that `chmod`/`chown` or write outside `$srcdir`/`$pkgdir`.
- An `.install` hook that downloads or executes anything.
- Disabled hardening on a C/C++ network-facing program.

## Related

- [[PKGBUILD Templates]] — the recipes this checklist guards
