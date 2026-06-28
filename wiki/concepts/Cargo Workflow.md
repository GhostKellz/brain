---
type: concept
title: "Cargo Workflow"
created: 2026-06-21
updated: 2026-06-28
aliases:
  - "Cargo"
  - "Cargo and rustc"
  - "rustc"
  - "rustup"
tags:
  - rust
  - cargo
  - toolchain
  - programming
status: developing
related:
  - "[[Rust]]"
  - "[[Rust Ownership and Borrowing]]"
---

# Cargo Workflow

The Rust toolchain has three layers: **`rustup`** (toolchain installer/manager),
**`rustc`** (the compiler), and **`cargo`** (the build system + package manager
you actually use day to day). You rarely call `rustc` directly — Cargo drives it.

> [!key-insight]
> `cargo` is the single front door: dependency resolution, building, testing,
> docs, linting, formatting, and publishing all run through one tool with one
> manifest (`Cargo.toml`) and one lockfile (`Cargo.lock`). This integrated
> toolchain is a big part of why Rust's developer experience is praised.

## rustup — toolchains and targets

```bash
rustup default stable           # set default channel (stable/beta/nightly)
rustup update                    # update installed toolchains
rustup toolchain install nightly
rustup component add clippy rustfmt rust-analyzer
rustup target add x86_64-unknown-linux-musl   # add a cross-compile target
rustup override set nightly      # pin this directory to a toolchain
```

A `rust-toolchain.toml` in a repo pins the channel/components so everyone builds
with the same compiler — analogous to how [[Zig]] repos pin `minimum_zig_version`.

## rustc — the compiler

`rustc` does the actual compiling, borrow-checking (see
[[Rust Ownership and Borrowing]]), monomorphisation, and LLVM codegen. You'll see
it surface mostly through Cargo, but direct use exists for one-offs:

```bash
rustc main.rs -O                 # compile a single file, optimised
rustc --edition 2021 main.rs
```

Editions (2015/2018/2021/2024) let the language evolve without breaking old code;
the edition is declared in `Cargo.toml`.

## cargo — the everyday driver

```bash
cargo new myapp                  # binary crate (--lib for a library)
cargo build                      # debug build → target/debug
cargo build --release            # optimised → target/release
cargo run -- <args>              # build + run
cargo test                       # run unit + integration + doc tests
cargo check                      # type-check without codegen (fast feedback)
cargo clippy                     # lint (catches idiom/correctness issues)
cargo fmt                        # format to rustfmt style
cargo doc --open                 # build + view API docs
cargo add serde --features derive   # add a dependency (edits Cargo.toml)
cargo update                     # bump deps within semver, update Cargo.lock
cargo install ripgrep            # build + install a binary crate to ~/.cargo/bin
```

### Manifest and lockfile
- **`Cargo.toml`** — declares package metadata, dependencies (with semver +
  features), and build profiles.
- **`Cargo.lock`** — pins the exact resolved dependency graph. **Commit it for
  binaries/apps; libraries may omit it.** `cargo build` respects it; `cargo
  update` changes it.

### Features
Conditional compilation flags that gate optional functionality/deps:

```toml
[features]
default = ["tls"]
tls = ["dep:rustls"]
```
```bash
cargo build --no-default-features --features tls
```

### Profiles
Build settings per mode — tune optimisation, LTO, debug symbols:

```toml
[profile.release]
opt-level = 3
lto = true
codegen-units = 1
strip = true
```

### Workspaces
Multiple crates sharing one `Cargo.lock` and `target/` — the standard layout for
multi-crate projects:

```toml
# top-level Cargo.toml
[workspace]
members = ["crates/*"]
resolver = "2"
```

## Publishing
```bash
cargo login <token>              # auth to crates.io
cargo publish --dry-run          # validate the package
cargo publish                    # ship to crates.io
```

> [!note] Useful add-ons: `cargo-watch` (rebuild on change), `cargo-nextest`
> (faster test runner), `cargo-edit` (now largely built-in), `cargo-audit`
> (RustSec vulnerability scan), `cargo-deny` (license/dep policy).

## Related

- [[Rust]] — the language overview
- [[Rust Ownership and Borrowing]] — what the compiler enforces
