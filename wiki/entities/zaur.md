---
type: entity
title: zaur
created: 2026-06-28
updated: 2026-06-28
tags:
  - zig
  - aur
  - arch
  - package-manager
  - repository
status: developing
entity_type: repository
language: Zig
repo_status: active
purpose: Zig-native AUR builder and repository server for self-hosted Arch Linux
  package management
related:
  - "[[Zig]]"
  - "[[zqlite]]"
  - "[[Arch Linux Administration]]"
  - "[[PKGBUILD Templates]]"
  - "[[Pacman Hooks]]"
  - "[[Self-Hosted Services]]"
  - "[[GhostKellz]]"
  - "[[CK Technology LLC]]"
---

# zaur

A Zig-native AUR builder and repository server for self-hosted Arch Linux
package management. Authored by [[GhostKellz]] / [[CK Technology LLC]],
MIT-licensed.

> [!key-insight]
> zaur both builds packages and serves them: it generates a pacman-compatible
> repo database and exposes it over HTTP, so a fleet can pull from one
> self-hosted repo.

## Stack

- **Language**: Zig (0.17.0-dev), ~10k lines.
- **Storage**: depends on [[zqlite]] v1.7.0.
- **Deployment**: systemd service template; Docker-based test suite.

## Capabilities

- AUR integration: PKGBUILD download plus `makepkg` builds.
- Multi-source builds: AUR, GitHub, and local sources.
- Pacman-compatible repo DB generation (`.db.tar.zst` / `.files.tar.zst`).
- Built-in HTTP server to serve repos directly to pacman.
- Arch mirror caching proxy: on-demand, with smart sync of databases only.
- PKGBUILD static-analysis scanner — detects pipe-to-shell, reverse shells,
  base64/eval obfuscation, setuid, suspicious writes, network fetches, and
  disabled checksums, each severity-scored.
- Auto PKGBUILD generation for Zig and Rust projects.
- GPG signing, REST API, build logging with per-device failure isolation, and
  dependency resolution.

## Status

v0.1.x, active development. Recent work: [[zqlite]] v1.7.0 upgrade, Zig 0.17
migration, and supply-chain hardening. MIT.

## Relationships

Built on [[Zig]] and backed by [[zqlite]]. Squarely in the
[[Arch Linux Administration]] domain — complements [[PKGBUILD Templates]] and
[[Pacman Hooks]] — and runs as one of the [[Self-Hosted Services]].
