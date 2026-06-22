---
type: reference
title: "tmux Cheatsheet"
created: 2026-06-21
updated: 2026-06-21
tags:
  - tmux
  - terminal
  - productivity
status: seed
related:
  - "[[Tiling Window Management]]"
---

# tmux Cheatsheet

**tmux** is a terminal multiplexer: multiple windows/panes inside one terminal,
plus **persistent sessions** that survive disconnects (essential over SSH). The
default prefix is `Ctrl+b`; commands are `prefix` then a key.

## Sessions (the killer feature)

```bash
tmux new -s work          # new named session
tmux attach -t work       # re-attach (e.g. after SSH drop)
tmux ls                   # list sessions
tmux kill-session -t work
tmux kill-server          # nuke everything
```

> [!key-insight]
> A tmux session keeps running on the host after you disconnect. Detach
> (`prefix d`), close your laptop, reconnect later, `tmux attach` — your shells,
> running processes, and layout are exactly as you left them.

## Windows (tabs)

| Action | Keys |
|--------|------|
| New window | `prefix c` |
| Rename window | `prefix ,` |
| Next / previous | `prefix n` / `prefix p` |
| List windows | `prefix w` |
| Jump to N | `prefix 1`…`9` |

## Panes (splits)

| Action | Keys |
|--------|------|
| Split vertical | `prefix %` |
| Split horizontal | `prefix "` |
| Cycle panes | `prefix o` |
| Show pane numbers | `prefix q` |
| Close pane | `exit` or `Ctrl+d` |

Resize from the command prompt (`prefix :`): `resize-pane -L 10` (also `-R/-U/-D`).

## Handy config tweaks

```tmux
set -g mouse on            # mouse: resize/select panes, scroll, copy
setw -g mode-keys vi       # vi-style copy mode

# Prefix-free pane navigation (vim-style, no Ctrl+b needed)
bind -n C-h select-pane -L
bind -n C-j select-pane -D
bind -n C-k select-pane -U
bind -n C-l select-pane -R

bind r source-file ~/.tmux.conf   # prefix r reloads config
```

> [!key-insight]
> Binding `Ctrl+h/j/k/l` to pane navigation (the `-n` flag = no prefix) makes
> moving between panes feel like a tiling WM — and matches vim split navigation,
> so the muscle memory is shared. See [[Tiling Window Management]].

## Copy mode (vi keys)

`prefix [` enters copy mode; navigate with `h j k l`, words with `w`/`b`, top/
bottom with `gg`/`G`. Search with `/` (forward), `?` (backward), `n`/`N` to repeat.

## Panic recovery

```bash
tmux ls                       # what's alive
tmux attach                   # attach to the most recent
tmux kill-session -t <name>   # drop a broken one
tmux kill-server && tmux      # full restart
```

## Related

- [[Tiling Window Management]] — the same split/navigate mental model at the WM level
