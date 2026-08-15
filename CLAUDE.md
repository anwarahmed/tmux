# CLAUDE.md

Working notes for this repo. Keep the **Work log** at the bottom current — append an entry
whenever a change lands, and correct the **Current state** section rather than letting it drift.

## What this repo is

A single-file tmux configuration (`tmux.conf`) that is checked out **directly into
`~/.config/tmux`** — the repo root *is* the live config directory. There is no install step
and no symlink: editing `tmux.conf` here edits the running config.

Reload after an edit with `prefix q` (bound to `source-file`), or `omarchy-restart-tmux`.

Files: `tmux.conf` (everything), `README.md` (install notes), `.gitignore`.

## The Omarchy relationship — read this before editing

The config is derived from [Omarchy's tmux bindings](https://learn.omacom.io/2/the-omarchy-manual/53/hotkeys#tmux),
and Omarchy actively rewrites it on the user's machine. Two mechanisms:

1. **Migrations** (`/usr/share/omarchy/migrations/*.sh`) — run on `omarchy-update`. They patch
   `~/.config/tmux/tmux.conf` in place with `sed`/append, guarded by `grep` checks. Past ones
   added the `M-Enter` pane splits, fixed `terminal-features[3]` → `set -ag`, and appended the
   OSC 52 clipboard line.
2. **`omarchy-refresh-tmux`** — calls `omarchy-refresh-config tmux/tmux.conf`, which
   **overwrites the file wholesale** with `/usr/share/omarchy/config/tmux/tmux.conf` and leaves
   the previous version at `tmux.conf.bak.<epoch>`. Local customizations are lost, not merged.

Practical consequences:

- Uncommitted local edits can be silently blown away. Commit before running `omarchy-update`.
- After any Omarchy update, `git diff` in this repo is the record of what changed — diff it
  against `/usr/share/omarchy/config/tmux/tmux.conf` to see whether the file is still stock.
- `*.bak.*` files are refresh leftovers and are gitignored. Diff them, then delete them.
- Deliberate personal divergence from the Omarchy default should be **documented below**, so
  it can be re-applied after a refresh clobbers it.

## Conventions

- Every binding carries a `-N "Description"` label. These are not cosmetic:
  `omarchy-menu-tmux-keybindings` parses them out of `tmux.conf` to build the searchable
  keybinding menu (`prefix ?`). A binding without `-N` is invisible in that menu.
- Flag order in the file is `bind -N "..." [-n|-T table] <key> <command>`.
- Bindings come in pairs: an Alt-prefixed root-table binding (`-n M-Enter`) and a
  prefix-table equivalent (`h`), often sharing the same description.
- Sections in order: Prefix → Config and help → Vi copy mode → Pane → Window → Session →
  General → Status bar → Theme. Theme colors are terminal palette names (`blue`,
  `brightblack`, `default`), never hex, so the active Omarchy theme drives them.

## Current state

`tmux.conf` is **byte-identical to the Omarchy default** at
`/usr/share/omarchy/config/tmux/tmux.conf` (verified 2026-08-15). There is currently no
personal divergence to protect.

Local environment: tmux 3.7b, Arch/Omarchy, kitty terminal.

### Known rough edges (pre-existing, not yet fixed)

- **Duplicate `escape-time`.** `tmux.conf:72` sets `set -g escape-time 0`; `tmux.conf:82` sets
  `set -sg escape-time 10`. Same server option, last write wins, so the effective value is 10ms
  and line 72 is dead. Both came from upstream Omarchy.
- **`prefix ?` is overridden.** It now opens the Omarchy keybinding popup instead of tmux's
  built-in `list-keys`. That popup shells out to `omarchy-menu-tmux-keybindings`, which only
  exists on Omarchy — on macOS (a documented target in `README.md`) the key produces an error
  popup. Built-in listing is still reachable via `:list-keys`.

### Deferred: macOS portability (audited 2026-08-15, no fix applied)

This config is used on macOS as well, where a large share of it is inert. Deferred by choice,
not yet addressed — 26 of the 42 bindings are Alt-based and at risk:

- **`extkeys` is granted to `xterm-kitty` only** (`tmux.conf:80`), so `extended-keys on` does
  nothing under any other TERM. That silently kills 13 bindings on macOS: `C-M-<arrow>` pane
  focus, `C-M-S-<arrow>` resize, `M-Enter`, `M-S-Enter`, `M-Escape`, `M-S-Left/Right`. Fix is
  to append the Mac terminal's TERM (`xterm-ghostty`, `xterm-256color` for iTerm2).
- **Option is not Meta by default on macOS terminals**, which disables the other 13
  (`M-1`..`M-9`, `M-<arrow>`). Per-terminal setting, not fixable from tmux.
- **`default-terminal "tmux-256color"`** (`tmux.conf:65`) may not exist in Apple's stock
  terminfo and can stop tmux from starting. Check with `infocmp tmux-256color`.
- **`C-Space` prefix** collides with the macOS "select previous input source" shortcut;
  Ctrl+arrows collide with Mission Control. `prefix2 C-b` is the working fallback.
- **`*:RGB`** (`tmux.conf:66`) misrenders on Terminal.app, which is not truecolor.
- **OSC 52 copy** is ignored by Terminal.app and needs a pref enabled in iTerm2;
  `copy-pipe-and-cancel "pbcopy"` is the macOS-native alternative.

The intended fix is an `if-shell '[ "$(uname)" = "Darwin" ]'` block. Note that such a block is
exactly what `omarchy-refresh-tmux` will erase, so record it here when it lands.

## Work log

- **2026-08-15** — Added this file and extended `.gitignore` to cover `.claude/` and `*.bak.*`.
  Committed the refreshed `tmux.conf` and deleted `tmux.conf.bak.1786746843` (its only unique
  content was one comment line; the setting it described survives at `tmux.conf:81`). Audited
  macOS portability — see Deferred above, no fix applied.
- **2026-08-14** — `omarchy-refresh-tmux` overwrote `tmux.conf` with the new Omarchy default
  (backup: `tmux.conf.bak.1786746843`). Net effect: `-N` descriptions on every binding, new
  `prefix ?` keybinding popup, and `set -as terminal-features ",*:clipboard"` folded in from the
  earlier migration (its explanatory comment was dropped). No functional keybinding changes.
- **2026-07-14** — `9b3d0df` pane shortcuts, kitty extended keys, window titles.
