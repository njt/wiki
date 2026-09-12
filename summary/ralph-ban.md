---
title: "ralph-ban"
url: https://github.com/kylesnowschwartz/ralph-ban
date_fetched: 2026-05-14
section: "AI-enhanced Coding"
topics:
  - agent-orchestration
---

# ralph-ban: Terminal Kanban Board

TUI kanban board implementing the "ralph loop" concept -- setting a target and working until completion. Built with Go, provides a terminal UI for project management with Claude Code integration.

## Key Features
- Five-column board: Backlog, To Do, Doing, Review, Done
- Vim-style navigation: `hjkl` keys for movement
- Real-time sync: Cards synchronize across CLI, Claude Code hooks, and TUI sessions via 2-second polling
- SQLite backing via beads-lite for persistence
- Claude Code integration with hooks and agents for automation
- Search with `/`, undo with `u`, priority with `+`/`-`

## Notable Quote
"A ralph loop sets a target and works until it's done. ralph-ban makes that target a kanban board."

## Usage Modes
- Interactive TUI for manual board management
- Batch mode with human approval for Claude Code automation
- Auto mode for unattended task processing
- Session continuation and resumption

Go (77.6%), Bubble Tea, SQLite, MIT license. Requires Go 1.25+.
