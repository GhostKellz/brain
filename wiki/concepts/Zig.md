---
type: concept
title: "Zig"
created: 2026-06-21
updated: 2026-06-28
tags:
  - language
  - systems-programming
  - zig
status: developing
related:
  - "[[Rust]]"
  - "[[Go]]"
  - "[[zqlite]]"
  - "[[Ghostlang]]"
  - "[[Grove]]"
  - "[[Ghost Ecosystem]]"
  - "[[NV Tools Suite]]"
---

# Zig

Zig is a general-purpose systems programming language and toolchain aimed at
robustness, optimality, and maintainability. It positions itself as a modern
alternative to C: no hidden control flow, no hidden memory allocations, manual
memory management with explicit allocators, comptime metaprogramming, and a
C/C++ cross-compiler bundled into the toolchain.

## Why it matters here

Zig is the primary "low-level" language across [[GhostKellz]]'s ecosystem. Where
[[Rust]] is used for application-level tooling and services, Zig is reached for
when the project wants zero external runtime dependencies, a small footprint,
explicit allocator control, or a C ABI surface for embedding.

## Key characteristics relevant to these projects

- **Explicit allocators** — every allocation passes an allocator, which makes
  memory-leak testing first-class (see [[zqlite]]'s leak-detection test suites).
- **comptime** — compile-time code execution used for generic data structures
  and configuration without macros.
- **C interop / C ABI** — Zig can both consume C headers and export a C ABI,
  enabling FFI bindings (e.g. [[zqlite]] C API, [[nvvk]] C ABI exports).
- **Single toolchain** — `zig build` drives compilation, testing, and
  cross-compilation; build logic is written in Zig itself (`build.zig`).
- **No package runtime deps** — `build.zig.zon` declares dependencies fetched by
  `zig fetch`; several projects deliberately ship with zero external packages.

> [!key-insight]
> The ecosystem tracks Zig **nightly/dev** releases (e.g. `0.17.0-dev`), not just
> stable tags. Project `build.zig.zon` files pin a `minimum_zig_version` to a
> specific dev build, so the toolchain version is part of each repo's contract.

## Modular builds — feature flags to shrink binaries

`build.zig` is real Zig code, so you can expose **build options** that gate
optional modules. Compiling out unused features (TLS backends, extra codecs,
debug tooling) keeps the binary small and the dependency graph minimal — the same
idea as Cargo features in [[Rust]] (see [[Cargo Workflow]]), but driven by
`b.option(...)` and `comptime`.

```zig
// build.zig
pub fn build(b: *std.Build) void {
    const target = b.standardTargetOptions(.{});
    const optimize = b.standardOptimizeOption(.{});

    // user-facing flags: zig build -Dtls=true -Dasync=false
    const enable_tls   = b.option(bool, "tls", "Enable TLS backend") orelse true;
    const enable_async = b.option(bool, "async", "Enable async runtime") orelse false;

    // surface them to the code via a generated Options module
    const opts = b.addOptions();
    opts.addOption(bool, "tls", enable_tls);
    opts.addOption(bool, "async", enable_async);

    const exe = b.addExecutable(.{
        .name = "app",
        .root_source_file = b.path("src/main.zig"),
        .target = target,
        .optimize = optimize,
    });
    exe.root_module.addOptions("build_options", opts);
    b.installArtifact(exe);
}
```

```zig
// src/main.zig — comptime-gate the feature so unused code is never compiled in
const opts = @import("build_options");
pub fn main() void {
    if (opts.tls) {
        // only pulled in when -Dtls=true
    }
}
```

```bash
zig build -Dtls=true -Dasync=false -Doptimize=ReleaseSmall
```

> [!key-insight]
> Because the flags are read at `comptime`, a disabled feature's code (and its
> imports) is **eliminated from the binary entirely** — not just skipped at
> runtime. Combine with `-Doptimize=ReleaseSmall` for the smallest artifact.

### Why people build this way

The goal is a library/tool that ships only what a given consumer actually uses,
so the same codebase can produce a fat full-featured build or a lean embedded one.

**Pros**
- **Smaller binaries** — unused backends/codecs/debug paths are never compiled in,
  which matters for embedded targets, WASM, and minimal containers ([[Docker]]).
- **Fewer dependencies pulled** — a feature that's off doesn't drag in its
  transitive packages, shrinking the supply-chain and build-time surface.
- **Faster builds** and simpler audits when you compile less.
- **One source, many configurations** — downstreams pick a profile
  (`-Dtls=false`) instead of forking the code.

**Cons / costs**
- **Combinatorial testing** — every flag combination is a build you should test;
  feature interactions can break in configs nobody compiles regularly.
- **`comptime` branching complexity** — heavy gating makes the code harder to read
  and reason about.
- **Documentation burden** — consumers need to know which flags exist and what
  they trade off.

This is the same trade-off as Cargo features in [[Rust]] (see [[Cargo Workflow]])
or build tags in [[Go]]: flexibility and size wins, paid for in test-matrix and
configuration complexity.

## Dependencies — `zig fetch --save`

`zig fetch` downloads a package, hashes it, and (with `--save`) records it in
`build.zig.zon` so builds are reproducible:

```bash
# pin to a release tag (recommended)
zig fetch --save https://github.com/ghostkellz/zsync/archive/refs/tags/v0.8.4.tar.gz

# track a moving branch (convenient, but not reproducible)
zig fetch --save https://github.com/ghostkellz/zsync/archive/refs/heads/main.tar.gz
```

This writes a `.dependencies` entry (URL + content hash) into `build.zig.zon`;
you then wire it into `build.zig` with `b.dependency("zsync", .{})` and
`exe.root_module.addImport(...)`.

### What `--save` actually does with the hash

`zig fetch` is content-addressed. When you run it:

1. **Download** the tarball from the URL to a temp location.
2. **Unpack** it and compute a hash over the *extracted package tree* — the file
   contents and layout — **not** the raw tarball bytes. (Re-compressing or
   re-tarballing the same files yields the same hash; this is why the hash is
   stable across mirrors.)
3. **Store** the unpacked package in the global cache at
   `~/.cache/zig/p/<hash>/`, keyed by that hash.
4. **`--save`** writes the entry into `build.zig.zon`:

```zig
// build.zig.zon
.dependencies = .{
    .zsync = .{
        .url = "https://github.com/ghostkellz/zsync/archive/refs/tags/v0.8.4.tar.gz",
        .hash = "zsync-0.8.4-<multihash>",   // the content hash zig computed
    },
},
```

On a later `zig build`, Zig looks for `<hash>` in the cache; if missing it
re-fetches from the URL, **re-hashes the extracted tree, and errors if the result
doesn't match** the recorded `.hash`. So the hash — not the URL — is the source of
truth: it makes builds reproducible and tamper-evident (a changed upstream tarball
fails verification instead of silently building different code).

> [!note] If you add the URL by hand without the hash, `zig build` prints the
> expected hash in the error; paste it into `build.zig.zon`. `zig fetch --save`
> just does that round-trip for you.

> [!key-insight]
> Prefer **version-pinned tag tarballs** (`refs/tags/vX.Y.Z`) over a branch
> archive. The recorded hash makes the build reproducible and tamper-evident; a
> `refs/heads/main` archive changes underneath you and breaks the hash on the next
> upstream commit.

## Strengths / trade-offs

**Strengths**
- No hidden control flow or allocations — what you read is what runs; explicit
  allocators make memory behaviour auditable and leak-testable.
- `comptime` replaces macros/generics with ordinary code run at compile time.
- Best-in-class C interop *and* a bundled C/C++ cross-compiler (`zig cc`).
- Tiny footprint, zero-runtime-dependency binaries; one `zig build` toolchain.

**Trade-offs**
- Pre-1.0: the language and stdlib still change between releases (this ecosystem
  tracks dev builds and pins `minimum_zig_version`).
- No borrow checker — memory safety is the programmer's job (unlike [[Rust]]);
  discipline + leak-detection tests carry the load.
- Smaller ecosystem than [[Rust]]/[[Go]]; fewer ready-made libraries.

## Projects in this vault built in Zig

- **Databases / storage**: [[zqlite]]
- **Languages / runtimes**: [[Ghostlang]], [[Grove]], [[GShell]], [[ghostls]]
- **GPU / graphics**: [[nvhud]], [[nvvk]], [[ghostVK]]
- **Infra / transport**: [[zquic]], [[zcrypto]], [[zsync]], [[Wraith]]
- **TUI / tooling**: [[Phantom]]

See [[Ghost Ecosystem]] and [[NV Tools Suite]] for the broader family maps. The
Rust-side counterparts are tracked under [[Rust]].
