---
type: reference
title: "Neovim Cheatsheet"
created: 2026-06-22
updated: 2026-06-22
tags:
  - neovim
  - editor
  - lua
  - productivity
status: seed
related:
  - "[[tmux Cheatsheet]]"
  - "[[Tiling Window Management]]"
---

This is the keymap reference for a personal **Neovim** config built on
[lazy.nvim](https://github.com/folke/lazy.nvim) — telescope, harpoon, flash,
LSP via mason, treesitter, and a stack of AI assistants. The **leader is
`Space`** and the **localleader is `\`**. Colorscheme is TokyoNight (moon,
transparent). The same split/navigate muscle memory as [[tmux Cheatsheet]]
applies via `vim-tmux-navigator`.

> [!key-insight]
> Mnemonic leader namespaces keep the map memorable: `<leader>f` = **find**
> (telescope), `<leader>h` = **harpoon**, `<leader>g` = **git**,
> `<leader>a` = **AI**, `<leader>x` = diagnostics (trouble), `<leader>c` = code.

## Files & search (Telescope)

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

Servers auto-installed via mason-lspconfig: `lua_ls`, `zls`, `pyright`,
`gopls`, `dockerls`, `yamlls`, `jsonls`, `bashls`; `rust_analyzer` via rustup
and `typescript-tools` for TS/JS.

## Diagnostics & symbols (Trouble / Outline)

| Action | Keys |
|--------|------|
| Toggle Trouble | `<leader>xx` |
| Workspace diagnostics | `<leader>xw` |
| Document diagnostics | `<leader>xd` |
| Toggle Outline | `<leader>o` |
| Toggle Undotree | `<leader>u` |

## Git

| Action | Keys |
|--------|------|
| Git status (Neogit) | `<leader>gs` |
| LazyGit | `<leader>gg` |
| Diffview open | `<leader>gd` |
| File history | `<leader>gh` |
| Close Diffview | `<leader>gq` |

## AI assistants

Copilot inline suggestions (insert mode):

| Action | Keys |
|--------|------|
| Accept suggestion | `<C-g>` |
| Next / previous suggestion | `<C-l>` / `<C-k>` |

Copilot Chat:

| Action | Keys |
|--------|------|
| Toggle chat | `<leader>aa` |
| Clear chat | `<leader>ax` |
| Quick prompt | `<leader>aq` |
| Prompt templates | `<leader>ap` |
| Submit (in chat buffer) | `<C-s>` |

`gen.nvim` against a **local** LLM endpoint (point it at your own
Ollama/LiteLLM base URL — keep that address out of any committed config):

| Action | Keys |
|--------|------|
| Select model | `<leader>ag` |
| Refactor | `<leader>ar` |
| Explain | `<leader>ad` |
| Complete | `<leader>ac` |
| Generate tests | `<leader>at` |

> [!key-insight]
> Inline Copilot accept is `<C-g>` (not Tab) so it never fights snippet/indent
> on Tab. AI verbs all live under `<leader>a`, so muscle memory carries across
> Copilot, gen.nvim, and the agent plugins.

## Debugging (DAP)

| Action | Keys |
|--------|------|
| Start / continue | `<F5>` |
| Step over / into / out | `<F10>` / `<F11>` / `<F12>` |
| Toggle breakpoint | `<leader>db` |

Adapters: `codelldb` (C/C++/Rust/Zig), `debugpy` (Python), `delve` (Go).

## Editor options worth knowing

`relativenumber` + `number` (hybrid line numbers), `clipboard=unnamedplus`
(system clipboard), `scrolloff=10`, 2-space `expandtab`, `wrap=false`,
`termguicolors`, and `modeline=false` (a deliberate hardening choice). Whitespace
is visible via `list`/`listchars`.

## Related

- [[tmux Cheatsheet]] — the multiplexer this editor lives inside; shared `C-h/j/k/l` navigation
- [[Tiling Window Management]] — same split/navigate mental model at the WM level
