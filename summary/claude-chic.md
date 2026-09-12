---
url: https://www.roborev.io/integrations/claudechic/
title: Claude Chic
author: Wes McKinney
date_fetched: 2026-05-15
date_published: 2026
topics:
  - agent-coding-workflow
---

# Claude Chic

An alternative terminal UI for Claude Code built on the Textual Terminal UI framework using the Claude Agent SDK. Created by Wes McKinney (mrocklin), MIT licensed, alpha status.

GitHub: https://github.com/mrocklin/claudechic

## Features

- Terminal UI for Claude Code sessions built with Python Textual framework
- Multi-agent support (running multiple agents concurrently)
- Git worktree integration for isolated parallel development
- Live roborev review sidebar with pass/fail verdict icons
- Animated spinners for in-progress reviews
- Auto-polls every 5 seconds for new results
- Click reviews to see detail inline
- Command palette with `/reviews`, `/reviews <job_id>`, `/reviewer [context]` commands
- `/roborev-fix` skill for batch-fixing open findings
- Sidebar filters by worktree/branch automatically

## Installation

- `uv tool install claudechic` (recommended)
- `pip install claudechic`

Prerequisites: roborev and claude (Claude Code) on PATH. For the `/roborev-fix` skill, run `roborev skills install` first.

## Workflow

1. Make changes in Claude Chic session
2. roborev reviews commits in background
3. Sidebar updates with verdict icons
4. Run `/reviews <job_id>` to read full review
5. Run `/roborev-fix` to batch-fix
6. Sidebar auto-updates with new verdict

The entire review-fix loop happens without switching windows or terminals.

## Architecture

Built on Claude Agent SDK + Textual TUI framework. Architecture docs at matthewrocklin.com/claudechic/ cover how the combination enables easy experimentation.

## See Also

- roborev: https://github.com/roborev-dev/roborev
- Claude Code
- Textual TUI framework
