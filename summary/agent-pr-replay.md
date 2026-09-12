---
title: "agent-pr-replay"
url: https://github.com/sshh12/agent-pr-replay
date_fetched: 2026-05-14
section: "Random"
topics:
  - agent-coding-workflow
  - guardrails-and-feedback-loops
---

# Agent PR Replay

Analyzes merged PRs from GitHub repositories to measure AI coding agent (Claude Code) performance vs human developers.

"The best way to improve an agent's ability to work in a codebase is to observe its default behavior, measure the gap against real human solutions, and steer it based on evidence."

Four-step process:
1. Ground Truth: Uses real merged PRs as validated human solutions
2. Replay Setup: Checks out repo at each PR's base commit, reverse-engineers task prompts from diff
3. Agent Execution: Runs Claude Code with identical prompts and starting conditions
4. Comparison & Synthesis: Identifies gaps, generates targeted guidance

Outputs:
- CLAUDE.md/AGENTS.md: Steering rules based on observed behavioral patterns
- skills.md: Reusable agent skills with structured YAML
- Statistics: Tool usage breakdowns, file access patterns, command analysis

Example findings from PyTorch analysis: "prefer deletion over defensive programming," "minimize changes to existing code structure."

Requirements: Python 3.11+, GitHub CLI, Claude Code CLI. ~$4 per analysis session. Run in trusted repos or sandboxed environments.
