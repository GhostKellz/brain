---
type: concept
title: "JavaScript"
created: 2026-06-28
updated: 2026-06-28
tags:
  - language
  - javascript
  - typescript
  - nodejs
  - programming
status: developing
related:
  - "[[Go]]"
  - "[[Python]]"
  - "[[Astro Static Site Generator]]"
  - "[[Alpine.js]]"
---

# JavaScript

JavaScript is the language of the web — dynamically typed, prototype-based, and
event-driven. The crucial distinction is **language vs runtime**: the JS *language*
(ECMAScript) is just a spec; what you actually run it on (browser, Node.js, Bun,
Deno) determines the APIs and tooling you get.

> [!key-insight]
> "JavaScript" the language hasn't changed much year-to-year, but the *runtimes*
> and *toolchains* churn constantly. Most real-world friction (npm, bundlers,
> module formats, TypeScript) lives in the runtime/tooling layer, not the language.

## Runtimes

| Runtime | What it is | Notes |
|---------|-----------|-------|
| **Browser** | the original host (V8/SpiderMonkey/JSC) | DOM, fetch, Web APIs |
| **Node.js** | server-side JS on V8 (2009) | the ecosystem standard; npm registry; `fs`/`http`/`crypto` |
| **Bun** | all-in-one runtime + toolkit (JavaScriptCore) | fast; *also* a package manager, bundler, test runner, and runs TS directly |
| **Deno** | secure runtime by Node's creator (V8) | TS-first, permissions model, web-standard APIs |

### Node.js
The default server runtime. Manage versions with **`nvm`** (or `fnm`/`volta`):

```bash
nvm install --lts && nvm use --lts
node --version
node server.js
node --watch server.js          # built-in watch mode (18+)
```

Node ships LTS (even majors, stable) vs Current. Modern Node has native `fetch`,
`--test` runner, and `.env` support — much of what once needed packages.

### Bun
A drop-in-ish Node alternative that's *also* the toolchain. Runs TypeScript and
JSX with no build step:

```bash
bun init                  # scaffold
bun install               # install deps (uses bun.lockb, very fast)
bun run dev               # run a script
bun test                  # built-in test runner
bun build ./index.ts      # built-in bundler
bun index.ts              # execute TS directly, no transpile step
```

> [!key-insight]
> Bun's pitch is "one tool instead of node + npm + tsc + jest + webpack." It reads
> `package.json`, so you can often `bun install` / `bun run` an existing Node
> project unchanged and get a big speed-up.

## npm and how dependencies work

```bash
npm init -y                     # create package.json
npm install express             # add a runtime dep (writes package-lock.json)
npm install -D typescript       # dev dependency
npm ci                          # clean install from the lockfile (CI/repro)
npm run build                   # run a "scripts" entry
npx <tool>                      # run a package binary without installing globally
```

How it behaves:
- **`package.json`** declares deps with semver ranges (`^1.2.0` = compatible
  minor/patch); **`package-lock.json`** pins the exact resolved tree.
- **`node_modules/`** is the (large, flattened) installed tree — always
  gitignored. `npm ci` wipes and rebuilds it from the lockfile for reproducible
  installs; `npm install` may update the lockfile.
- **Semver caret/tilde**: `^` allows minor+patch, `~` allows patch only, exact
  pins drop the prefix.

### npm vs the alternatives

| Manager | Why | Lockfile |
|---------|-----|----------|
| **npm** | bundled with Node, universal | `package-lock.json` |
| **pnpm** | content-addressed store + symlinks → fast, disk-efficient, strict | `pnpm-lock.yaml` |
| **yarn** | workspaces, PnP (yarn 2+) | `yarn.lock` |
| **bun** | fastest installs, integrated runtime | `bun.lockb` |

> [!note] Pick one per repo and commit its lockfile. `pnpm` is the common choice
> for monorepos; `bun` if you want runtime + manager unified; `npm` for zero
> surprises.

## TypeScript

Typed superset of JS that compiles (or is stripped) to plain JS. The de-facto
default for non-trivial projects — types catch errors before runtime and power
editor autocomplete.

```bash
npm i -D typescript && npx tsc --init    # creates tsconfig.json
npx tsc                                   # type-check + emit JS
```

Bun/Deno run `.ts` directly; Node needs `tsc`, `tsx`, or `--experimental-strip-types`.

## Module systems

- **ESM** (`import`/`export`) — the modern standard; enable with
  `"type": "module"` in `package.json` or `.mjs`.
- **CommonJS** (`require`/`module.exports`) — legacy Node default; `.cjs`.
  Interop friction between the two is a perennial gotcha.

## Frameworks / libraries worth knowing

| Area | Picks |
|------|-------|
| Server | **Express**, **Fastify**, **Hono** (edge/runtime-agnostic), **NestJS** |
| Frontend | **React**, **Vue**, **Svelte**, **Solid**; **[[Alpine.js]]** for sprinkles |
| Meta-frameworks | **Next.js**, **Nuxt**, **SvelteKit**, **[[Astro Static Site Generator]]** |
| Build/bundle | **Vite** (default dev/build), **esbuild**, **Rollup**, **Turbopack** |
| Lint/format | **ESLint**, **Prettier**, **Biome** (Rust-based, fast all-in-one) |
| Test | **Vitest**, **Jest**, **Playwright** (e2e) |
| Validation | **Zod** (schema + inferred types) |

## Strengths / trade-offs

**Strengths**
- Runs everywhere (browser + server + edge); one language full-stack.
- Largest package ecosystem (npm registry); huge talent pool.
- Async I/O model (event loop) is excellent for high-concurrency I/O servers.
- TypeScript makes large codebases maintainable.

**Trade-offs**
- Toolchain churn and config sprawl (bundlers, module formats, transpilers).
- Dynamic typing footguns without TypeScript; `node_modules` bloat.
- Single-threaded by default (worker threads / clustering for CPU parallelism).
- ESM/CJS interop and dependency-tree security (supply chain) need attention.

## Related

- [[Go]] · [[Python]] — the other general-purpose languages here
- [[Astro Static Site Generator]] · [[Alpine.js]] — the web-dev usage in this vault
