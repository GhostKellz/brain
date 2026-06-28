---
type: concept
title: "Go"
created: 2026-06-28
updated: 2026-06-28
tags:
  - language
  - go
  - programming
status: developing
related:
  - "[[Rust]]"
  - "[[Zig]]"
  - "[[Python]]"
  - "[[JavaScript]]"
  - "[[Docker]]"
---

# Go

Go (Golang) is a statically typed, compiled, garbage-collected language from
Google built for **simplicity, fast builds, and easy concurrency**. It compiles
to a single static binary, has a famously small spec, and ships a batteries-
included toolchain (`go build`, `go test`, `gofmt`, `go vet`) in one binary.

> [!key-insight]
> Go's design constraint is *readability at scale*: one obvious way to do things,
> a tiny syntax, mandatory formatting (`gofmt`), and a fast compiler. It trades
> expressive power (no macros, historically no generics) for a codebase any
> teammate can read on day one. That's why the cloud-native stack — Docker,
> Kubernetes, Terraform, Prometheus, etcd — is written in Go.

## Where it fits vs Rust / Zig

The two-language thesis here is **[[Rust]] for apps, [[Zig]] for engines**. Go
sits beside them as the **ops / cloud-native / CLI glue** language:

| | Go | [[Rust]] | [[Zig]] |
|--|----|----|----|
| Memory | GC | ownership/borrow | manual + allocators |
| Concurrency | goroutines + channels | async/await + Send/Sync | manual |
| Compile speed | very fast | slow | fast |
| Runtime deps | none (static binary) | none | none |
| Best at | CLIs, services, cloud tooling | safety-critical apps, daemons | low-level engines, C ABI |
| Learning curve | gentle | steep | moderate |

Reach for Go when you want a service or CLI shipped fast, cross-compiled to a
static binary, with concurrency that's hard to get wrong.

## Concurrency model

Go's headline feature: lightweight **goroutines** (cheap green threads, ~2 KB
stacks) communicating over **channels**.

```go
func main() {
    ch := make(chan string)
    go func() { ch <- "done" }()   // goroutine
    fmt.Println(<-ch)              // blocks until received
}

// fan-in with select
select {
case v := <-ch1:
    handle(v)
case <-time.After(time.Second):
    log.Println("timeout")
}
```

> [!key-insight]
> "Don't communicate by sharing memory; share memory by communicating." Channels
> are the idiomatic synchronisation primitive, but `sync.Mutex`, `sync.WaitGroup`,
> `sync/atomic`, and `errgroup` are all there when a mutex is genuinely simpler.
> The `context.Context` type threads cancellation/deadlines through call trees.

## Toolchain (all the `go` stuff)

One binary does everything; no external build system.

```bash
go mod init github.com/you/proj   # create go.mod (module path)
go get github.com/spf13/cobra     # add a dependency (updates go.mod/go.sum)
go mod tidy                        # prune + add missing deps
go build ./...                     # compile everything
go run .                           # build + run
go test ./...                      # run all tests
go test -race ./...                # with the race detector
go vet ./...                       # static checks
gofmt -w .                         # canonical formatting (or `go fmt`)
go install github.com/x/y@latest   # build + drop binary in $GOBIN
```

- **Modules** (`go.mod` / `go.sum`) — versioned dependencies with a checksum
  database; `GOPROXY` caches the module graph. No lockfile/vendor needed (though
  `go mod vendor` exists).
- **Cross-compile for free** — `GOOS=linux GOARCH=arm64 go build` produces a
  static binary for another platform with no toolchain install.
- **Workspaces** (`go.work`) — multi-module local development.
- Common third-party linters: `golangci-lint` (aggregates `staticcheck`,
  `govet`, `errcheck`, etc.).

## The libraries worth knowing

### CLI / TUI (the Charm + spf13 stack)
- **`cobra`** — the de-facto CLI framework (commands, subcommands, flags); powers
  `kubectl`, `gh`, `hugo`. Scaffolds with `cobra-cli`.
- **`viper`** — config management (flags + env + files + remote KV), pairs with
  `cobra` for 12-factor config precedence.
- **`pflag`** — POSIX/GNU-style flags (cobra's flag layer).
- **Charm suite** — `bubbletea` (Elm-architecture TUI framework), `bubbles`
  (ready-made TUI widgets: lists, spinners, text inputs), `lipgloss` (styling/
  layout), `glamour` (markdown rendering), `huh` (forms), `log` (pretty logger).

### Web / services
- **`net/http`** — the stdlib HTTP server is production-grade on its own.
- **`chi`** / **`gin`** / **`echo`** / **`fiber`** — routers/frameworks; `chi`
  is stdlib-idiomatic, `gin`/`echo` are batteries-included, `fiber` is fasthttp.
- **`grpc-go`** + `protobuf` — RPC; **`connect-go`** for gRPC/HTTP duality.

### Data / infra
- **`sqlx`** / **`pgx`** (Postgres) / **`sqlc`** (codegen from SQL) / **`gorm`**
  (ORM) / **`ent`** (schema-as-code ORM).
- **`zap`** / **`zerolog`** — fast structured logging; `slog` is now in the stdlib.
- **`testify`** — assertions/mocks; **`gomock`** for interface mocks.
- **`prometheus/client_golang`** — metrics (see [[Prometheus Monitoring]]).

## Strengths / trade-offs

**Strengths**
- Single static binary, trivial cross-compilation, fast cold-start — ideal for
  containers ([[Docker]]) and CLIs.
- Goroutines make concurrent I/O code straightforward.
- Fast compiles + mandatory `gofmt` = low-friction large teams.
- Rich stdlib (HTTP, crypto, JSON, templates) — many projects need few deps.

**Trade-offs**
- GC means it's not for hard-real-time or the lowest-latency engine work (that's
  [[Zig]] / [[Rust]]).
- Error handling is verbose (`if err != nil` everywhere) by design.
- Generics arrived late (1.18) and are deliberately limited.
- The opinionated simplicity can feel restrictive coming from richer languages.

## Related

- [[Rust]] · [[Zig]] — the systems-language counterparts
- [[Python]] · [[JavaScript]] — the other general-purpose languages here
- [[Docker]] — Go binaries are the canonical minimal-container payload
