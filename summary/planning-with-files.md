---
title: "Planning With Files"
url: https://github.com/OthmanAdi/planning-with-files
date_fetched: 2026-05-14
section: "AI-enhanced Coding"
topics:
  - agent-memory-and-context
---

# Planning with Files: Manus-Style AI Agent Workflow

Claude Code skill implementing persistent markdown planning -- the pattern attributed to Manus AI (acquired by Meta for $2B in December 2025).

## Core Problem
AI agents suffer from volatile memory, goal drift, hidden errors, and context stuffing. TodoWrite disappears after context resets.

## The 3-File Pattern
- `task_plan.md` -- phases and progress tracking
- `findings.md` -- research and discoveries
- `progress.md` -- session logs and test results

**Fundamental Principle:** "Context Window = RAM (volatile, limited); Filesystem = Disk (persistent, unlimited)"

## Key Features
- Automatic plan re-reading before decisions (via PreToolUse hooks)
- Progress reminders after file writes
- Completion verification before stopping
- Session recovery after context clearing

## Manus Principles
- Filesystem-based memory persistence
- Attention manipulation through plan re-reading
- Error logging for future reference
- Goal tracking via checkboxes

## Results
96.7% pass rate (29/30 assertions with skill vs 6.7% without). 100% blind A/B superiority.

Supports 17+ platforms. Available in 6 languages.
21.1k GitHub stars.
