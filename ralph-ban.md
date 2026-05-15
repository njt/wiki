# ralph-ban

TUI kanban board for agents, built in Go with Bubble Tea. Five columns (Backlog, To Do, Doing, Review, Done), vim-style navigation, real-time sync across CLI/Claude Code hooks/TUI sessions via 2-second polling, SQLite backing. "A ralph loop sets a target and works until it's done. ralph-ban makes that target a kanban board."

---

## Key Quotes

> "A ralph loop sets a target and works until it's done. ralph-ban makes that target a kanban board."

## Key Themes

#task-management #kanban #tui #claude-code #agents

The "ralph loop" concept is simple and effective: set a target, work until completion. The kanban board gives that loop visual structure and persistent state. The Claude Code integration (hooks + agents) means the board isn't just a display -- it's an active coordination surface where agents can claim and move cards.

Three usage modes: interactive TUI (manual management), batch mode (human approval for agent automation), and auto mode (unattended processing). The progression from human-in-the-loop to autonomous is well-designed.

Part of the same ecosystem as [[workgraph]] (which provides the more sophisticated task graph), [[weft]] (Cloudflare-hosted alternative), and [[Managing Agents via Kanban Boards]] (Geoffrey Litt's approach). ralph-ban is the simplest of these -- a kanban board and nothing more, which might be exactly the right level of complexity for most projects.

## Critical Analysis

Sometimes the right tool is the one with the fewest features. ralph-ban doesn't try to be a task graph, an orchestration system, or a project management tool. It's a kanban board with agent integration, and that's enough. The 2-second polling for sync is pragmatic but will cause issues at scale -- SQLite can handle it, but the polling interval means cards can briefly exist in inconsistent states across sessions. For 1-3 agents on a personal project, this is ideal. For anything bigger, look at [[workgraph]].

---
*Sources: [[raw/ralph-ban]]*
*Last updated: 2026-05-14*
