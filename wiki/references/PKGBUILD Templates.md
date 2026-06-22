---
type: reference
title: "PKGBUILD Templates"
created: 2026-06-21
updated: 2026-06-21
tags:
  - arch-linux
  - packaging
  - pkgbuild
  - aur
status: seed
related:
  - "[[PKGBUILD Auditing]]"
  - "[[Pacman Hooks]]"
---

# PKGBUILD Templates

A **PKGBUILD** is the build recipe pacman/`makepkg` uses to produce an Arch
package. Maintaining a per-language template — modelled on the official
`PKGBUILD-vcs.proto` — means shipping your own projects becomes "copy, fill three
placeholders, build" instead of writing a recipe each time.

## Build + install flow

```sh
cp packaging/<lang>/PKGBUILD ~/build/myproj/PKGBUILD
cd ~/build/myproj
$EDITOR PKGBUILD                       # fill placeholders
makepkg --printsrcinfo > .SRCINFO      # sanity-check metadata
makepkg -si                            # build + install
```

Common placeholders: `NAME` (package/binary name), `REPO` (upstream path), and
`pkgdesc`, plus the real `depends`/`makedepends`.

## `-git` (VCS) packages

A `-git` PKGBUILD clones the repo and derives the version from git history, so it
builds **without a release tag**:

```sh
# with tags:    git describe → 1.2.3.r4.gabcdef
# before tags:  r<commit-count>.<short-sha>
```

Both forms increase monotonically under `vercmp`, so upgrades resolve correctly.

> [!key-insight]
> Follow the upstream idiom and derive the source-tree name from
> `${pkgname%-git}` rather than inventing a `_pkgname` variable — one source of
> truth for the name.

## Switching to a tagged-release tarball

When a project starts cutting releases and you want reproducible checksums:

1. Drop the `-git` suffix and delete `pkgver()`.
2. Set `pkgver` to the release version.
3. Point `source` at the archive:
   `source=("$pkgname-$pkgver.tar.gz::$url/archive/v$pkgver.tar.gz")`
4. Replace `sha256sums=('SKIP')` with real sums (`updpkgsums` / `makepkg -g`).

## Per-language notes

| Language | Key options |
|----------|-------------|
| **Rust** | `cargo fetch --locked` in `prepare()` for offline reproducible builds; needs a committed `Cargo.lock`. |
| **Go** | Distro `CGO_*`/`GOFLAGS` for a hardened trimmed PIE; `options=('!lto')` (Go owns linking); `-ldflags "-X main.version=$pkgver"`. |
| **Zig** | `-Doptimize=ReleaseSafe`, `-Dcpu=baseline` for portability; `options=('!buildflags' '!lto')` (Zig owns flags/linking). |
| **Node** | `arch=('any')`; `npm ci` against the lockfile; `npm install -g --prefix "$pkgdir/usr"`. |
| **Bun** | `bun build --compile` → single self-contained binary; `options=('!strip')` (stripping breaks the embedded runtime). |

## Where packaging lives in a repo

| Location | Use when |
|----------|----------|
| `packaging/` | Default for code projects — packaging files only |
| `release/` | Packaging is part of a broader release workflow |
| repo root | Trivial / AUR-only projects with a single PKGBUILD |

Keep it in a subfolder so `makepkg` artifacts (`src/`, `pkg/`, tarballs) don't
pollute the repo root.

## Related

- [[PKGBUILD Auditing]] — reviewing a recipe before you build it
- [[Pacman Hooks]] — `.install` scripts are a related per-package mechanism
