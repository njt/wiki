---
title: "napkin"
url: https://github.com/blader/napkin
date_fetched: 2026-05-14
section: "AI-enhanced Coding"
topics:
  - agent-memory-and-context
---

# Napkin: Persistent Memory for Claude Code

A skill for Claude Code that enables persistent memory across sessions through a per-repository markdown scratchpad (`.claude/napkin.md`). Allows AI agents to learn from mistakes and improve performance over multiple sessions.

## Key Features
1. **Persistent Learning**: Agent maintains a markdown file tracking errors, corrections, and successful approaches across sessions
2. **Continuous Logging**: Documents mistakes, user corrections, environment discoveries, and preferences as they occur during work
3. **Progressive Improvement**: Performance noticeably improves by sessions 3-5, with the agent anticipating issues before correction needed
4. **Per-Repo Design**: One napkin file per repository; can be committed (shared with contributors) or ignored (personal use)

## What Gets Logged
- Agent's own mistakes and false assumptions
- User corrections and feedback
- Environment-specific surprises
- User preferences for task execution
- Successful approaches worth repeating

## Description
"Baby continual learning in a markdown file." Incremental knowledge accumulation that compounds over time rather than one-time learning.

529 stars, MIT license.
