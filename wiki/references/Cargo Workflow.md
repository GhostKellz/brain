---
type: reference
title: "Cargo Workflow"
created: 2026-06-21
updated: 2026-06-21
tags:
  - rust
  - cargo
  - tooling
status: seed
related:
  - "[[Rust Ownership and Borrowing]]"
  - "[[PKGBUILD Templates]]"
---

# Cargo Workflow

**Cargo** is Rust's build system and package manager. It handles dependencies,
building, testing, formatting, linting, docs, and benchmarks through one CLI.

## Daily commands

```bash
rustup update            # update the toolchain
cargo fmt                # format (rustfmt)
cargo check              # type/borrow-check without producing a binary (fast)
cargo build              # debug build
cargo build --release    # optimized build
cargo run                # build + run
cargo test               # run unit + integration tests
```

> [!key-insight]
> `cargo check` runs the full compiler front-end (including the borrow checker)
> but skips codegen — it's the fast inner loop for catching errors. Save
> `cargo build` for when you actually need to run the binary.

## CI-grade linting

```bash
cargo clippy --all-targets --all-features -- -D warnings
```

`clippy` is Rust's linter; `-D warnings` turns every warning into an error so CI
fails on lint regressions. Pair with `cargo fmt --check` to enforce formatting.

## Dependencies

```bash
cargo add <crate>        # add a dependency
cargo rm <crate>         # remove one
cargo update             # update within semver constraints
cargo tree               # show the dependency graph
```

## Docs, benches, examples

```bash
cargo doc --open                 # build + open API docs
cargo bench                      # benchmarks (e.g. criterion)
cargo run --example <name>       # run an examples/ binary
cargo clean                      # remove target/
```

## Reproducible builds and `Cargo.lock`

For **binaries/apps**, commit `Cargo.lock` so builds are reproducible. This is also
what makes offline, locked packaging work:

```bash
cargo fetch --locked     # fetch exactly the locked versions (offline-prep)
cargo build --frozen     # build offline, fail if the lockfile is stale
```

This `--locked` / `--frozen` pattern is exactly how a Rust [[PKGBUILD Templates|PKGBUILD]]
gets reproducible, offline `build()`/`check()` phases.

## Project config

- `.cargo/config.toml` (project) or `~/.cargo/config.toml` (global) for things
  like the linker, target dirs, and `git-fetch-with-cli`.
- Consider `#![deny(warnings)]` in `lib.rs`/`main.rs` for strictness.
- `rust-analyzer` in the editor for completions and inline diagnostics.

## Related

- [[Rust Ownership and Borrowing]] — the language model `cargo check` enforces
- [[PKGBUILD Templates]] — packaging Rust binaries on Arch
