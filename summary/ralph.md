---
title: "Ralph"
url: https://github.com/snarktank/ralph
date_fetched: 2026-05-14
section: "LLMs"
topics:
  - coding-agents-and-frameworks
  - agent-architecture
---

# Ralph: Autonomous AI Agent Loop

The "Wiggum loop" -- forces the agent to iterate through product requirements until all PRD items are complete. 19k stars, MIT licensed, primarily TypeScript.

## Core Architecture

Each iteration creates a new AI context window with clean state. Continuity via:
- Git commit history from previous runs
- progress.txt (learnings appended across iterations)
- prd.json (tracks story completion status)

## Workflow Loop

1. Read highest-priority incomplete story from prd.json
2. Spawn fresh AI instance with prompt + codebase context
3. Implement single story
4. Execute quality checks (typecheck, tests)
5. Commit changes if checks pass
6. Mark story passes: true in prd.json
7. Append findings to progress.txt
8. Repeat until all stories complete or iteration limit reached

## Key Components

| File | Function |
|------|----------|
| ralph.sh | Bash orchestrator (--tool amp or --tool claude) |
| prompt.md / CLAUDE.md | Tool-specific prompts |
| prd.json | Machine-readable task list with completion flags |
| progress.txt | Append-only knowledge base for future iterations |

## Critical Design Patterns

- Right-Sized Stories: Tasks must fit single context windows
- AGENTS.md Documentation: Each iteration updates docs with discovered patterns
- Feedback Loops Required: typecheck, tests, CI catch errors immediately
- Browser Verification: Frontend stories invoke dev-browser skill

## Notable Features

- Automatic context handoff when Amp fills
- Archive system preserving previous runs by branch/date
- Interactive flowchart visualization
- Support for both Amp and Claude Code
