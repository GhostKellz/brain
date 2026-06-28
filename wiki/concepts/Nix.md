---
type: concept
title: "Nix"
created: 2026-06-28
updated: 2026-06-28
aliases:
  - "NixOS"
  - "Nix Flakes"
  - "nixpkgs"
  - "home-manager"
  - "nix-darwin"
tags:
  - nix
  - nixos
  - reproducibility
  - declarative
  - packaging
  - devops
status: developing
related:
  - "[[Docker]]"
  - "[[Linux Administration]]"
  - "[[Arch Linux Administration]]"
  - "[[Btrfs Snapshots]]"
  - "[[Ansible]]"
---

# Nix

**Nix** is four things wearing one name: a **purely functional package manager**,
a lazy **functional language** for describing builds, a **build system**, and —
as **NixOS** — a whole Linux distribution configured by a single expression. The
unifying idea is that *every* build is a deterministic function of its declared
inputs, so the same expression yields the same artifact on any machine, forever.

> [!key-insight]
> Most package managers mutate shared global state — `/usr/lib`, `/etc`,
> a single version of each library — so "works on my machine" is a coin flip.
> Nix never mutates: every package lands in `/nix/store/<hash>-name/` where the
> hash is derived from **all** of its inputs (source, deps, compiler flags, the
> compiler itself). Change any input → new hash → new path. Nothing is ever
> overwritten, so installs don't conflict, upgrades can't half-apply, and a
> rollback is just pointing a symlink at the previous hash. Reproducibility isn't
> a feature bolted on; it's the data model.

## The store is the whole trick

`/nix/store` is an immutable, content-addressed-ish content tree. A path like
`/nix/store/abcd1234…-openssl-3.3.2/` is named by a hash over the **derivation**
that built it. Consequences fall straight out of this:

- **No dependency hell.** Two apps needing different OpenSSL versions get two
  store paths; neither sees the other. There is no single "system OpenSSL."
- **Atomic upgrades & rollback.** Your environment is a *generation* — a symlink
  pointing at a set of store paths. Switching generations is one symlink swap;
  rolling back is swapping it the other way. No partial state.
- **Multi-user, unprivileged installs.** Users build into the shared store
  (via the `nix-daemon`) without root and without stepping on each other.
- **Perfect GC.** `nix-collect-garbage` deletes any store path not reachable from
  a live generation (a "GC root"). Nothing leaks, nothing is guessed.

```bash
nix-store -q --references $(which hello)   # exact runtime closure of a binary
nix path-info -rsh nixpkgs#firefox          # full closure + sizes
nix-collect-garbage -d                       # delete old generations + dead paths
```

## Derivations: builds as pure functions

A **derivation** is the build recipe Nix actually executes: a `.drv` file listing
inputs, the builder, the environment, and outputs. Nix realises it in a **sandbox**
with **no network** and only the declared inputs on `PATH` — that sandbox is what
*forces* reproducibility (you can't accidentally depend on something undeclared).
The hash of the derivation's inputs becomes the output path, so identical inputs
→ identical path → Nix can **substitute** a prebuilt binary from a cache
(`cache.nixos.org`) instead of rebuilding.

> [!key-insight]
> This is why "build once, run anywhere in the org" actually works: a CI box and
> a laptop compute the *same* output hash from the same flake lock, so the laptop
> just downloads the CI-built artifact from the binary cache. The cache is keyed
> by the reproducibility, not the other way around.

## The language

Nix-the-language is a small, lazy, functional, dynamically-typed expression
language. You rarely write much of it day to day, but the shape matters:

```nix
{ pkgs ? import <nixpkgs> {} }:

pkgs.mkShell {
  packages = with pkgs; [ ripgrep fd jq python313 ];
  shellHook = ''echo "dev shell ready"'';
}
```

- **Lazy**: nothing evaluates until needed — `nixpkgs` is a ~100k-package
  attribute set you can reference cheaply because only the paths you touch build.
- **Pure**: no side effects; an expression's value depends only on its inputs.
- **Everything is an expression** — `nixpkgs` itself is one giant expression that
  evaluates to the package set.

## Flakes — the modern entry point

A **flake** is a directory with a `flake.nix` (typed `inputs` + `outputs`) and a
`flake.lock` that pins every input to an exact git revision + hash. It's the unit
that finally gave Nix **hermetic, version-pinned, composable** projects — the lock
file is the reproducibility guarantee made concrete.

```nix
{
  inputs.nixpkgs.url = "github:NixOS/nixpkgs/nixos-26.05";
  outputs = { self, nixpkgs }: {
    devShells.x86_64-linux.default =
      nixpkgs.legacyPackages.x86_64-linux.mkShell {
        packages = [ /* … */ ];
      };
  };
}
```

```bash
nix develop          # enter the dev shell defined by this flake
nix build .#foo      # build an output, pinned by flake.lock
nix flake update     # bump inputs, rewrite the lock
nix run nixpkgs#hello # run a package with zero install
```

> [!note] Flakes are *de facto* universal but officially still an **experimental**
> feature upstream. The ecosystem has split: **upstream Nix** keeps the
> experimental label, **Determinate Nix** declared flakes stable (and ships
> exclusive `lazy trees` for fast evaluation), and the community fork **Lix**
> consolidated a de-facto flakes v1. All three share the same nixpkgs, NixOS,
> home-manager, and nix-darwin — you pick an implementation, not an ecosystem.

## NixOS — the OS as one expression

NixOS applies the store model to the *entire system*. `configuration.nix` (or a
flake's `nixosConfigurations`) declares packages, services, users, kernel,
bootloader, firewall — everything — and `nixos-rebuild switch` realises it.

```nix
{ config, pkgs, ... }: {
  networking.hostName = "workstation";
  services.openssh.enable = true;
  users.users.alice = { isNormalUser = true; extraGroups = [ "wheel" ]; };
  environment.systemPackages = with pkgs; [ git neovim ];
  system.stateVersion = "26.05";
}
```

```bash
sudo nixos-rebuild switch          # build + activate the new generation
sudo nixos-rebuild boot            # stage for next boot, don't switch now
sudo nixos-rebuild switch --rollback   # go back a generation
nixos-rebuild build-vm             # boot the WHOLE config in a throwaway VM
```

> [!key-insight]
> Every `switch` creates a new **generation** that appears as a **boot-menu
> entry**. A broken config (bad kernel, wrong driver, dead service) is recovered
> by rebooting into the previous generation from the bootloader — the OS-level
> equivalent of a [[Btrfs Snapshots|filesystem snapshot]], except it's the
> *configuration* that's versioned, not just the files. You can also `build-vm`
> a config to test it in a VM before ever touching the real machine.

**channels vs flakes** is the one fork in NixOS workflow:
- **Channels** — the classic mutable pointer (`nixos-rebuild` follows the
  `nixos-26.05` channel; `nix-channel --update` moves it). Simple, less hermetic.
- **Flakes** — `flake.lock` pins the exact nixpkgs revision; `nixos-rebuild switch
  --flake .#hostname`. Reproducible across every machine in a fleet. The modern
  default for anyone managing more than one host.

## Cross-platform: home-manager & nix-darwin

The same declarative model reaches user dotfiles and macOS:

- **home-manager** — declares a *user's* environment (packages, dotfiles, shell,
  services) as Nix. Runs standalone on any Linux distro (not just NixOS), or as a
  NixOS/nix-darwin module. This is how you get [[Zsh and Powerlevel10k|your shell]],
  editor, and tools reproducible on a non-Nix host.
- **nix-darwin** — does for macOS what NixOS does for Linux: system-level config,
  packages, and defaults, declaratively. Pairs with home-manager for user config.
  (Note: x86_64-darwin support ends after nixpkgs 26.05; Apple Silicon only going
  forward.)

A common pattern is one flake holding `nixosConfigurations`,
`darwinConfigurations`, and `homeConfigurations` for every machine you own —
the entire personal fleet in one version-pinned repo. This is the niche
[[ghostctl]] nods to with its NixOS rebuild/rollback helpers.

## Nix vs Docker — the comparison that matters

Both promise reproducibility, but at *different layers*, and they compose rather
than compete.

| | Docker | Nix |
|--|--------|-----|
| **Unit** | An image (a tarball of layers) | A derivation → store path (a build graph) |
| **Reproducibility** | The *image* is fixed; the *Dockerfile* often isn't (`apt-get` pulls "latest", `:latest` base drifts) | The *build* is fixed — same inputs, same output hash, bit-for-bit |
| **How it isolates** | Runtime: namespaces/cgroups isolate a running container | Build time: a sandbox isolates the build; runtime deps are the exact closure |
| **Granularity** | Whole-OS image; rebuild a layer → rebuild everything above | Per-package; change one dep → only its closure rebuilds, rest is cached |
| **Image size** | Often bloated (full base distro per image) | Minimal closures — ship only the exact runtime deps |
| **Dev shells** | `docker run -it` into a container | `nix develop` — tools on PATH, no container, no daemon |
| **Layer caching** | Cache invalidates top-down by Dockerfile order | Content-addressed: shared store paths dedup across *all* builds |

> [!key-insight]
> A `Dockerfile` describes *how to mutate* a base image step by step; its
> reproducibility ends the moment `apt-get update` hits the network or a `:latest`
> base shifts under you. Nix describes *what the result is* as a pure function of
> pinned inputs — so two builds a year apart from the same `flake.lock` are
> identical. Docker reproduces the *result* (the frozen image); Nix reproduces the
> *process* (the build itself). That's why "it built in CI but not locally" is a
> Docker-era problem Nix structurally removes.

**They compose.** Nix is one of the best ways to *build* Docker images:
`pkgs.dockerTools.buildLayeredImage` produces minimal, reproducible OCI images
with no Dockerfile and optimal layer sharing — then ship them to any
[[Docker]]/[[Kubernetes]] runtime. Use Nix for the *what*, your container runtime
for the *where it runs*.

## Where Nix really shines

- **Dev environments** — `nix develop` (or [`direnv` + `nix-direnv`]) gives every
  repo its exact toolchain, pinned, with zero "install these 14 things first."
  New contributor clones, `direnv allow`, done.
- **Fleet management** — one flake, many hosts, all on a pinned nixpkgs. Declarative
  config management with atomic rollback, in the same spirit as [[Ansible]] but
  with reproducibility and rollback built into the substrate, not layered on top.
- **CI reproducibility** — the binary cache means CI builds once and every machine
  fetches the identical artifact; "works in CI" *guarantees* "works locally."
- **Reproducible images** — minimal, deterministic OCI images via `dockerTools`.
- **Disaster recovery** — your whole machine is `git clone` + `nixos-rebuild`.

## Pros / trade-offs

**Pros**
- Reproducibility as a structural guarantee, not a discipline.
- Atomic upgrades + instant rollback at package *and* whole-OS level.
- No dependency conflicts; multi-version coexistence by design.
- One declarative model spans dev shells → user config → whole OS → containers.

**Trade-offs**
- Steep learning curve — the Nix language and the store model are genuinely new.
- Docs historically fragmented (improving: `nix.dev`, the official wiki).
- The flakes "experimental" status + the upstream/Determinate/Lix split is real
  ecosystem friction; pick an implementation deliberately.
- Some software resists the model (FHS-assuming binaries) and needs wrappers
  (`buildFHSEnv`, `steam-run`, `nix-ld`).
- Store can grow large between GC runs — schedule `nix-collect-garbage -d`.

## Related

- [[Docker]] — the container runtime Nix complements (and can build images for)
- [[Btrfs Snapshots]] — the filesystem-level analogue to NixOS generations/rollback
- [[Ansible]] — imperative-ish config management; Nix is the declarative counterpart
- [[Arch Linux Administration]] · [[Linux Administration]] — the rolling/imperative model Nix inverts
- [[ghostctl]] — sysadmin toolkit with NixOS rebuild/rollback helpers
