---
url: https://addyosmani.com/blog/loop-engineering/
title: Loop Engineering
author: Addy Osmani
date_fetched: 2026-06-15
date_published: 2026-06-07
---

# Loop Engineering

by Addy Osmani, June 7, 2026

## Core Definition

Loop engineering: instead of personally writing prompts for coding agents, you design systems that prompt agents autonomously. A loop is a "recursive goal where you define a purpose and the AI iterates until complete."

## Key Quotes

- "Loop engineering is replacing yourself as the person who prompts the agent."
- "You shouldn't be prompting coding agents anymore. You should be designing loops that prompt your agents." — Peter Steinberger
- "I don't prompt Claude anymore. I have loops running that prompt Claude" — Boris Cherny

## The Five Loop Components + State

### 1. Automations
Scheduled tasks that perform discovery and triage independently. Codex: Automations tab with configurable projects, prompts, cadence, environment. Claude Code: /loop, /goal, cron tasks, hooks, GitHub Actions. Both support a /goal primitive that keeps executing until a verifiable condition is met.

### 2. Worktrees
Separate working directories on their own branches sharing the same repo history, ensuring parallel agents don't collide. Codex: built-in worktree support per thread. Claude Code: git worktree, --worktree flag, isolation: worktree setting on subagents.

### 3. Skills
SKILL.md-based format that codifies project conventions, build steps, and tribal knowledge so agents don't rediscover context each session. "An agent starts every session cold" and will fill intent gaps with confident guesses unless skills are present.

### 4. Plugins and Connectors
Built on MCP (Model Context Protocol) to connect agents to issue trackers, databases, APIs, and Slack. The difference between "an agent that says 'here is the fix' and a loop that opens the PR, links the Linear ticket and pings the channel."

### 5. Sub-agents
Separating the writer from the reviewer. A second agent with different instructions evaluates what the first produced. Codex: sub-agents as TOML in .codex/agents/. Claude Code: .claude/agents/ and agent teams. "The model that wrote the code is way too nice grading its own homework."

### 6. State/Memory
A markdown file or Linear board that persists across runs. "The model forgets everything between runs so the memory has to be on disk."

## Example Loop Architecture

An automation runs daily, calling a triage skill that reads CI failures, open issues, and recent commits. Findings are written to markdown or Linear. For each actionable item, an isolated worktree spawns a sub-agent to draft a fix and a second sub-agent to review it against project skills and tests. Connectors open the PR and update the ticket. The state file tracks what was tried and what remains.

## Three Problems That Sharpen

1. **Verification is still on the engineer** — unattended loops make unattended mistakes. Even with a verifier sub-agent, "done" is a claim, not proof.
2. **Comprehension debt grows** — the faster a loop ships code the engineer didn't write, the wider the gap between what exists and what is understood.
3. **Cognitive surrender** — "designing the loop is the cure when you do it with judgement and the accelerant when you do it to avoid thinking."

## Closing

Prompting agents directly remains effective. Two people can build identical loops and get opposite results: one moves faster on work they deeply understand; the other uses it to avoid understanding entirely. "Build the loop. But build it like someone who intends to stay the engineer, not just the person who presses go."
