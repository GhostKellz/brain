---
type: reference
title: "Neovim Cheatsheet"
created: 2026-06-22
updated: 2026-06-28
tags:
  - neovim
  - editor
  - lua
  - lazyvim
  - productivity
status: developing
related:
  - "[[tmux Cheatsheet]]"
  - "[[Tiling Window Management]]"
  - "[[Go]]"
  - "[[Rust]]"
  - "[[Python]]"
---

# Neovim Cheatsheet

Keymaps **and** the full plugin/package setup for a personal **Neovim** config
built directly on [lazy.nvim](https://github.com/folke/lazy.nvim) as the package
manager (not the LazyVim distro — see below). The **leader is `Space`**, the
**localleader is `\`**, and the colorscheme is **TokyoNight (moon, transparent)**.
Split/navigate muscle memory is shared with [[tmux Cheatsheet]] via
`vim-tmux-navigator`.

> [!key-insight]
> Mnemonic leader namespaces keep the map memorable: `<leader>f` = **find**
> (telescope), `<leader>h` = **harpoon**, `<leader>g` = **git**, `<leader>a` =
> **AI**, `<leader>x` = diagnostics (trouble), `<leader>c` = code, `<leader>o` =
> outline/opencode. Learn the prefix, and which-key shows the rest.

## lazy.nvim vs LazyVim vs the rest

These names get conflated constantly — they are different things:

| Thing | What it is |
|---|---|
| **lazy.nvim** | the **plugin manager** (by folke) — lazy-loading, lockfile, UI. This config uses it directly |
| **LazyVim** | a full **distro** *built on top of* lazy.nvim — opinionated defaults, preconfigured plugins. Not used here, but the same `:Lazy` mechanics apply |
| **packer.nvim** | the older plugin manager lazy.nvim largely replaced (now unmaintained) |
| **kickstart.nvim** | a single-file *starter* config you read and grow yourself — great for learning the from-scratch approach this config takes |

This config is the "**roll your own with lazy.nvim**" path: a small `init.lua`
plus a `lua/plugins/` folder where each plugin is its own spec file.

## Config layout

```
~/.config/nvim/
├── init.lua              # bootstrap lazy.nvim, leader keys, core UI, theme
└── lua/
    ├── config/           # non-plugin config, loaded via pcall (safe)
    │   ├── options.lua    # vim.opt settings
    │   ├── navigation.lua # motion keymaps (+ flash)
    │   ├── formatting.lua # tabstop/shiftwidth/expandtab
    │   ├── lsp.lua        # LSP servers (modern vim.lsp.start API)
    │   ├── autocmds.lua   # yank highlight, etc.
    │   ├── dap.lua        # debug adapters
    │   └── patch.lua      # runtime shims for deprecated plugin APIs
    └── plugins/          # one file per plugin → imported as { import = "plugins" }
```

`init.lua` bootstraps the manager (clone if missing), then
`require("lazy").setup({ { import = "plugins" }, ... })` pulls every spec from
`lua/plugins/`. Config modules load through a `tryreq()` pcall wrapper so a
single broken file never hard-crashes startup.

## Plugin management (lazy.nvim)

| Command | Action |
|---|---|
| `:Lazy` | open the manager UI (status of every plugin) |
| `:Lazy sync` | install missing + update + clean removed (the everyday one) |
| `:Lazy update` | update plugins, write `lazy-lock.json` |
| `:Lazy restore` | pin all plugins back to the lockfile commits |
| `:Lazy clean` | remove plugins no longer in the specs |
| `:Lazy profile` | startup profiler (find slow plugins) |
| `:Lazy log` | recent commits pulled per plugin |

- **`lazy-lock.json`** pins every plugin to an exact commit — commit it to dotfiles
  for reproducible installs across machines (`:Lazy restore` to match).
- **Lazy-loading** is per-spec: `event = "VeryLazy"`, `cmd = ...`, `keys = ...`,
  `ft = ...`, or `event = "InsertEnter"` keep startup fast.
- LuaRocks/hererocks are disabled (`rocks = { enabled = false }`) to cut noise.

## Tooling install (Mason) & parsers (Treesitter)

LSP servers, DAP adapters, and formatters are *binaries* — managed by **Mason**,
separate from plugin management:

| Command | Action |
|---|---|
| `:Mason` | Mason UI — install/update/remove tool binaries |
| `:MasonUpdate` | refresh the Mason registry |
| `:TSUpdate` | update Treesitter parsers |
| `:TSInstall <lang>` | install a parser |
| `:checkhealth` | diagnose providers, LSP, treesitter, clipboard |

`mason-lspconfig` auto-installs: `lua_ls`, `zls`, `pyright`, `gopls`, `dockerls`,
`yamlls`, `jsonls`, `bashls`. Rust uses **rustup's** toolchain-matched
`rust-analyzer` (not Mason's) so it tracks the active toolchain; TS/JS uses
`typescript-tools`. Treesitter parsers: lua, bash, json, yaml, markdown, vim,
python, go, rust, zig, toml, html, css, javascript, typescript.

## The plugin stack

Grouped by job — every entry is an actual spec in `lua/plugins/`:

| Area | Plugins |
|---|---|
| **Manager** | `lazy.nvim` |
| **Theme/UI** | `tokyonight.nvim` (moon, transparent), `mini.statusline`, `noice.nvim` + `nvim-notify`, `fidget.nvim` (LSP progress), `nvim-web-devicons`, `nvim-colorizer.lua`, `indent-blankline.nvim` |
| **Files/nav** | `neo-tree.nvim`, `telescope.nvim` (+ `fzf-native`, `ui-select`), `harpoon` (harpoon2), `vim-tmux-navigator` |
| **LSP/tooling** | `nvim-lspconfig`, `mason.nvim` + `mason-lspconfig`, `schemastore.nvim`, `typescript-tools.nvim`, `none-ls.nvim`, `fidget.nvim` |
| **Completion** | `nvim-cmp` (+ `copilot-cmp`) or `blink.cmp` (+ `blink-cmp-copilot`), `LuaSnip` + `friendly-snippets`, `nvim-autopairs` |
| **Treesitter** | `nvim-treesitter` (main branch / new API) |
| **Git** | `gitsigns.nvim`, `neogit` (+ `diffview.nvim`), `lazygit.nvim` |
| **Diagnostics** | `trouble.nvim`, `outline.nvim`, `todo-comments.nvim`, `undotree` |
| **Testing/Debug** | `neotest` (+ `-python`/`-go`/`-rust`/`-vitest`/`-plenary`), `nvim-dap` (+ `dap-ui`, `dap-virtual-text`) |
| **Editing** | `Comment.nvim`, `flash` (motion, optional), `mini.*` |
| **AI** | `copilot.lua` + `CopilotChat.nvim`, `gen.nvim` (local LLM), `claude-code.nvim`, `opencode.nvim` (+ `snacks.nvim`), `edgy.nvim` |

## Files & search (Telescope)

Telescope uses the **ivy** bottom-split theme with fzf-native sorting.

| Action | Keys |
|--------|------|
| Find files | `<leader>ff` |
| Live grep | `<leader>fg` |
| Open buffers | `<leader>fb` |
| Recent files | `<leader>fr` |
| Help tags | `<leader>fh` |
| Commands | `<leader>fc` |
| Keymaps | `<leader>fk` |
| LSP document symbols | `<leader>fs` |
| LSP workspace symbols | `<leader>fw` |
| Toggle file tree (Neo-tree) | `<leader>e` |

## Harpoon (pinned files)

| Action | Keys |
|--------|------|
| Add current file | `<leader>ha` |
| Toggle quick menu | `<leader>hm` |
| Jump to slot 1 / 2 / 3 | `<leader>h1` / `h2` / `h3` |

> [!key-insight]
> Harpoon beats a fuzzy finder for the 3–4 files you bounce between all day —
> pin them once, then `<leader>h1`/`h2`/`h3` is an instant jump with no picker.

## Motion (flash + fast moves)

| Action | Keys | Mode |
|--------|------|------|
| Flash jump | `s` | normal / visual / operator |
| Flash treesitter select | `S` | normal / visual / operator |
| Remote flash | `r` | operator |
| Start / end of line | `H` / `L` | normal |
| Jump 5 lines down / up | `J` / `K` | normal |
| Arrow keys in insert | `<A-h/j/k/l>` | insert |

(Flash keymaps are `pcall`-guarded — they no-op if flash isn't installed.)

## LSP

| Action | Keys |
|--------|------|
| Go to definition | `gd` |
| Go to references | `gr` |
| Hover docs | `K` |
| Signature help | `<C-k>` |
| Rename symbol | `<leader>rn` |
| Code action | `<leader>ca` |
| Line diagnostics (float) | `<leader>d` |

The LSP layer uses the **modern `vim.lsp.start` API** (no
`require('lspconfig')`), with per-filetype autocmds, format-on-save, and inlay
hints. `none-ls` adds prettier/stylua/black/gofmt as external formatters.

## Diagnostics & symbols (Trouble / Outline)

| Action | Keys |
|--------|------|
| Toggle Trouble | `<leader>xx` |
| Workspace diagnostics | `<leader>xw` |
| Document diagnostics | `<leader>xd` |
| Toggle Outline | `<leader>o` / `<leader>cs` |
| Toggle Undotree | `<leader>u` |

Virtual text is **off** by default (signs + underline + float on demand) to keep
the buffer clean.

## Git

| Action | Keys |
|--------|------|
| Git status (Neogit) | `<leader>gs` |
| LazyGit | `<leader>gg` |
| Diffview open | `<leader>gd` |
| File history | `<leader>gh` |
| Close Diffview | `<leader>gq` |

`gitsigns` adds the gutter signs + inline blame; Neogit and Diffview are
integrated (Neogit opens diffs in Diffview).

## Testing (neotest)

`neotest` with python/go/rust/vitest/plenary adapters. Drive via commands or
map your own:

```vim
:lua require('neotest').run.run()          " nearest test
:lua require('neotest').run.run(vim.fn.expand('%'))  " file
:lua require('neotest').summary.toggle()   " summary panel
```

## Debugging (DAP)

| Action | Keys |
|--------|------|
| Start / continue | `<F5>` |
| Step over / into / out | `<F10>` / `<F11>` / `<F12>` |
| Toggle breakpoint | `<leader>db` |

Adapters auto-detected: `codelldb` (C/C++/Rust/Zig, Mason), `debugpy` (Python),
`delve` (Go). For Rust/Zig/C: build first, hit `<F5>`, it guesses
`target/debug/<crate>` or prompts for the binary.

## AI assistants

Copilot inline suggestions (insert mode):

| Action | Keys |
|--------|------|
| Accept suggestion | `<C-g>` or `<M-l>` |
| Next / previous | `<C-l>`·`<M-]>` / `<C-k>`·`<M-[>` |

Copilot Chat:

| Action | Keys |
|--------|------|
| Toggle chat | `<leader>aa` |
| Clear chat | `<leader>ax` |
| Quick prompt | `<leader>aq` |
| Prompt templates | `<leader>ap` |
| Submit (in chat buffer) | `<C-s>` |

`gen.nvim` against a **local** LLM endpoint (point it at your own Ollama/LiteLLM
base URL — keep that address out of any committed config):

| Action | Keys |
|--------|------|
| Select model | `<leader>ag` |
| Refactor | `<leader>ar` |
| Explain | `<leader>ad` |
| Complete | `<leader>ac` |
| Generate tests | `<leader>at` |

Agent plugins (terminal-embedded):

| Action | Keys |
|--------|------|
| Toggle Claude Code | `<leader>ac` / `<C-,>` |
| Claude continue / verbose | `<leader>cC` / `<leader>cV` |
| Toggle OpenCode | `<leader>ao` |
| OpenCode ask / add context | `<leader>oa` / `<leader>op` |

> [!key-insight]
> Inline Copilot accept is `<C-g>` (not Tab) so it never fights snippet/indent
> on Tab. AI verbs cluster under `<leader>a`, so muscle memory carries across
> Copilot, gen.nvim, Claude Code, and OpenCode.

## Theming

TokyoNight **moon**, **transparent** background, with heavy `on_highlights`
overrides (mint-green functions/normal, hacker-blue comments, yellow numbers) and
a custom `mini.statusline`. `nvim-colorizer` previews hex colors inline;
`indent-blankline` draws indent guides.

## Editor options worth knowing

`relativenumber` + `number` (hybrid line numbers), `clipboard=unnamedplus`
(system clipboard, set via `vim.schedule` to not slow startup), `scrolloff=10`,
`cursorline`, 2-space `expandtab`, `wrap=false`, `termguicolors`, visible
whitespace (`list`/`listchars`), and **`modeline=false`** — a deliberate
hardening choice (modeline parsing has been an RCE vector).

## Troubleshooting

| Symptom | Fix |
|---|---|
| Plugin won't load / errors on startup | `:Lazy` → check status; `:Lazy sync`; read `:messages` |
| LSP not attaching | `:LspInfo` / `:checkhealth lsp`; confirm the server installed in `:Mason`; check root markers (`.git`, `Cargo.toml`, `go.mod`, …) |
| Treesitter highlight broken after update | `:TSUpdate`; on the new main-branch API re-run `:checkhealth nvim-treesitter` |
| Formatter not running | server must support formatting *or* a `none-ls` source exists for the filetype |
| Completion missing Copilot | `:Copilot status` / `:Copilot auth`; ensure `copilot-cmp`/`blink-cmp-copilot` loaded |
| DAP can't find adapter | `:Mason` → install `codelldb`/`debugpy`/`delve` |
| "deprecated" spam | provider/host warnings — set `python3_host_prog`/`node_host_prog`; `:checkhealth provider` |
| Broke after a plugin update | `:Lazy restore` pins back to `lazy-lock.json` |
| Slow startup | `:Lazy profile` to find the offender; add `event`/`cmd`/`keys` lazy triggers |

> [!note]
> Pin reproducibility to `lazy-lock.json`: update deliberately with `:Lazy sync`,
> test, then commit the lockfile. `:Lazy restore` is the undo button when an
> update breaks something mid-week.

## Related

- [[tmux Cheatsheet]] — the multiplexer this editor lives inside; shared `C-h/j/k/l` navigation
- [[Tiling Window Management]] — same split/navigate mental model at the WM level
- [[Go]] · [[Rust]] · [[Python]] — the languages this LSP/DAP stack targets
