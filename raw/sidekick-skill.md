---
url: https://github.com/jleechanorg/claude-commands/blob/main/.claude/skills/sidekick/SKILL.md
title: Sidekick Skill — Persistent Worker Agent for Claude Code
author: jleechanorg
date_fetched: 2026-07-18
date_published: 2025-2026
---

# Sidekick Skill

A Claude Code skill for spawning persistent, crash-recoverable worker agents (sidekicks) that handle long-running missions. The skill uses STATE.md checkpoints and a resumption bead for durability, with tmux-based process persistence as a fallback for sessions that must survive the parent CLI exiting.

## Two Modes

**Default: In-session teammate.** The sidekick runs as a named, background `Agent` within the invoking session's Agent Team. It's visible in the user's panel and SendMessage-addressable both ways. Durability comes from STATE.md checkpoints and a resumption bead — not process persistence. If the conversation crashes, a fresh session respawns from disk state.

**Fallback: External tmux session.** Used only when work must survive the session ending or when Agent Teams is unavailable. The sidekick runs in `tmux new-session` as a real Claude Code (`-p` or interactive TUI) or Codex process. The user must be told "it cannot appear in the team panel" and given the attach command.

**Engine support:** Both Claude (Sonnet default) and Codex engines, with detailed launch commands for each. For Claude: `-p` (headless) or interactive TUI mode. For Codex: `codex exec` with appropriate flags.

## Invocation

`/sidekick [sonnet|codex] [mission...]`

Default model is Sonnet.

## Core Mechanics

### STATE.md

Located at `/tmp/<project-slug>/sidekick/<mission-slug>/STATE.md`. Contains:

- **Mission:** What this sidekick is doing
- **Ground truth:** Non-negotiable invariants
- **Standing rules:** Operational constraints
- **Progress Log:** Append-only record of actions taken
- **Next Actions:** Rewritten each step with what to do next

### Checkpoint Cadence

Every 5 minutes, each supervisor and lane owner must:
1. Append a timestamped heartbeat to STATE.md
2. Update the resumption bead
3. Commit state so "a crash loses ≤5 min"

### Safe Commits

If the working tree has unrelated changes, route commits to an isolated state repo or use path-scoped `git add`.

## Auth & Environment

Auth env propagation is MANDATORY in the launch command. The tmux server snapshots environment at startup, so inline `CLAUDE_CONFIG_DIR` and any routing overrides into the tmux command string. Never trust the tmux server to pass environment variables.

## Communication Channels

1. **Main session → sidekick:** STATE.md + tmux (this IS the channel, not a fallback)
2. **Inside the sidekick:** Attempt team-based lanes but verify they formed before reporting them as such
3. **Main session's visible team:** Spawned via the session's own Agent tool — these die with the parent CLI

Agent Teams is one-team-per-session with no cross-session join. Sidekicks running in external tmux sessions are NOT SendMessage-addressable from the main session.

## Team Visibility Mandate

The user always wants the sidekick visible in the invoking session's Agent Team panel. The hybrid pattern requires in-process teammates as default, disk-based durability, and tmux only as fallback.

## Milestone Reporting

When reporting milestones, the main session polls the tmux pane and STATE.md. Reports require proof: "tmux capture, git status/log, PR/commit URL, or test output."

## Completion Detection and Lane Results

After a milestone report, the main session determines whether the mission is complete. Lane results are collected via tmux capture-pane or STATE.md entries and synthesized into a final report.

## Stall Watchdog (Interactive TUI — Mandatory)

Interactive TUI sidekicks fail by **stalling alive**, not by exiting. The session-limit modal blocks all work silently.

**Watchdog rules:**
- Watch STATE.md mtime (>15 min = alarm)
- Check EVERY teammate pane (not just the lead)
- Recover via `tmux send-keys -t <pane> Enter`

The document documents real incidents from 2026-07-10 and 2026-07-11 that informed the design, including auth env propagation failures, quota-exhaustion stalls, and team visibility issues.

## Drive-Loop Invariants

Three rules from a PR incident retro:
1. **Endgame single-writer freeze:** When a lane reaches the final implementation stage, only one writer may touch the target files
2. **External-bot requests are once-per-head:** Never re-request a review or CI check that's already pending
3. **Background waits need a deadman:** Any wait for an external process must carry a timeout and notify on expiry

## Migration Between Modes

The skill defines procedures to migrate an in-session teammate to an external tmux session (when the parent session is dying) and vice versa (when a new session picks up an orphaned tmux sidekick). The STATE.md file is the invariant across mode transitions.

## Engine-Specific Details

### Claude Engine
- `-p` flag for headless mode
- Interactive TUI for supervised work
- Team formation differences between modes

### Codex Engine
- `codex exec` command structure
- Different team formation behavior
- Environment propagation specifics
