---
title: "How Boris Uses Claude Code"
url: https://threadreaderapp.com/thread/2007179832300581177.html
date_fetched: 2026-05-14
section: "AI-enhanced Coding"
topics:
  - agent-coding-workflow
---

# How Boris Uses Claude Code - Boris Cherny

Boris Cherny, creator of Claude Code, shares his personal setup and best practices. He emphasizes that his "surprisingly vanilla" approach demonstrates Claude Code's flexibility rather than prescribing a single correct methodology.

## Parallel Session Management

Cherny runs 5 Claude instances simultaneously in his terminal (numbered tabs 1-5) with system notifications, plus 5-10 additional instances on claude.ai/code and the iOS app. He frequently teleports sessions between platforms and hands off work between local and web environments.

## Model Selection

He exclusively uses "Opus 4.5 with thinking for everything" despite its larger size, reasoning that superior steering and tool-use capabilities make it faster overall than smaller models requiring more correction.

## Institutional Knowledge

Teams maintain CLAUDE.md files checked into git repositories, where the team collectively documents Claude's errors and adds preventive instructions. This allows Claude to learn team-specific patterns across sessions.

## Planning and Execution

Sessions typically begin in Plan mode (shift+tab twice), enabling back-and-forth refinement of Claude's approach before switching to auto-accept edits mode for implementation.

## Slash Commands

Custom commands stored in .claude/commands/ automate frequent workflows. Example: `/commit-push-pr` uses inline bash to pre-compute git status, reducing model back-and-forth.

## Subagents

Specialized agents like code-simplifier and verify-app automate post-generation tasks, approximating the "most common workflows" across pull requests.

## Permission Management

Rather than using `--dangerously-skip-permissions`, Cherny pre-allows safe bash commands via `/permissions`, with common configurations shared team-wide in .claude/settings.json.

## Tool Integration via MCP

Claude Code integrates with external systems through MCP servers, enabling:
- Slack posting and searching
- BigQuery queries for analytics
- Sentry error log retrieval
- Chrome extension testing for UI verification

## Verification

Cherny identifies verification as "probably the most important thing to get great results" from Claude Code, claiming it can "2-3x the quality of the final result."

His verification approach includes:
- Chrome extension-based browser testing
- Automated UI interaction and iteration
- Test suite execution
- Domain-specific validation methods

For long-running operations, he uses verification agents, Stop hooks, or the ralph-wiggum plugin to ensure quality without requiring constant monitoring.

## Code Quality

A PostToolUse hook handles code formatting after generation, addressing the remaining 10% of formatting concerns to prevent CI failures.

## Key Quote

"We intentionally build it in a way that you can use it, customize it, and hack it however you like. Each person on the Claude Code team uses it very differently."
