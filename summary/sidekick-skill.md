---
url: https://github.com/jleechanorg/claude-commands/blob/main/.claude/skills/sidekick/SKILL.md
title: "Sidekick Skill — Persistent Worker Agent for Claude Code"
author: jleechanorg
date_fetched: 2026-07-18
date_published: 2025-2026
topics:
  - agent-architecture
---

A Claude Code skill for spawning persistent, crash-recoverable worker agents ("sidekicks") that handle long-running missions. The default mode runs the sidekick as a named background `Agent` inside the invoking session's Agent Team, using `STATE.md` checkpoints and a resumption bead for durability — if the session crashes, a fresh one respawns from disk state. As a fallback, the sidekick can run in an external `tmux` session when work must survive the parent CLI exiting entirely; this mode sacrifices team-panel visibility.

The sidekick checkpoints progress to `STATE.md` every five minutes: a file at `/tmp/<project-slug>/sidekick/<mission-slug>/STATE.md` holding the mission description, ground-truth invariants, standing rules, an append-only progress log, and the next actions list. Each heartbeat appends a timestamp and updates the resumption bead, capping data loss at five minutes.

Authentication environment variables must be propagated explicitly in the launch command — `tmux` servers snapshot environment at startup and will not inherit them automatically. Communication from the main session to the sidekick goes through `STATE.md` plus `tmux capture-pane`; Agent Teams is one-team-per-session with no cross-session joins, so sidekicks in external `tmux` sessions are not `SendMessage`-addressable.

Interactive TUI sidekicks have a documented failure mode of stalling alive (typically from the session-limit modal) rather than exiting. A stall watchdog monitors `STATE.md` mtime (>15 minutes triggers an alarm) and checks every teammate pane, recovering stalled panes via `tmux send-keys`. Three drive-loop invariants emerged from incident retro: an endgame single-writer freeze on target files, once-per-head external-bot requests, and deadman timeouts on background waits. The skill defines procedures to migrate sidekicks between in-session and external-tmux modes, with `STATE.md` as the invariant across transitions. Both Claude and Codex engines are supported, with detailed launch commands for each.
