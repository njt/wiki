---
url: https://github.com/kylesnowschwartz/ralph-ban
date_fetched: 2026-07-05
backfilled: true
---

A ralph loop sets a target and works until it's done. ralph-ban makes that target a kanban board.

Terminal kanban built with Bubble Tea, backed by beads-lite's SQLite database. Five columns (Backlog, To Do, Doing, Review, Done) with vim-style navigation. The board pans horizontally when the terminal is too narrow. Cards sync across the `bl` CLI, Claude Code hooks, and other TUI sessions via 2-second polling.

My mate said I should have named this Gas-Ban, but it's not nearly that weird. Planner helps break down the tasks `ralph-ban claude --plan`. Orchestrator delegates to workers, reviewers, and oracles `ralph-ban claude`. Keep Orchestrator working until ToDo/Doing/Review are moved to Done `ralph-ban claude --auto`. That's about it.

Requires Go 1.25+.

`go install github.com/kylesnowschwartz/ralph-ban@latest`Or build from source:

```
git clone git@github.com:kylesnowschwartz/ralph-ban.git
cd ralph-ban
go build .
```
Initialize a project, then open the board:

```
ralph-ban init --demo    # create project with a demo board (Conway's Game of Life)
ralph-ban                # open the board
```
ralph-ban ships a Claude Code plugin with hooks and agents. `ralph-ban init` extracts them into `.ralph-ban/plugin/` and `.claude/agents/`.

```
ralph-ban claude                    # batch mode (pauses for human merge approval)
ralph-ban claude --auto             # drain the board without intervention
ralph-ban claude --continue         # continue most recent session
ralph-ban claude --resume           # interactive session picker
```
`?` toggles full help. `Ctrl+z` suspends the TUI (resume with `fg`).

**Navigation**

| Key | Action | 
|---|---|
| `h`/`←` | Focus left column | 
| `j`/`↓` | Move cursor down | 
| `k`/`↑` | Move cursor up | 
| `l`/`→` | Focus right column | 
| `/` | Search cards | 

**Cards**

| Key | Action | 
|---|---|
| `n` | New card | 
| `e` | Edit card | 
| `d` | Delete (press twice to confirm) | 
| `z` | Zoom card detail | 
| `Enter` | Move card right | 
| `Backspace` | Move card left | 
| `u` | Undo last move | 
| `+`/`-` | Change priority | 

Requires just for task running. Uses a `go.work` workspace with `../beads-lite` for local development.

```
just build     # build
just test      # run tests
just run       # build and launch TUI
just lint      # vet + staticcheck
just release   # tag, push, create GitHub release
```
MIT
