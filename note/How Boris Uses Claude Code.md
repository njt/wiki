# How Boris Uses Claude Code

Boris Cherny -- the creator of Claude Code -- describes his personal workflow in a Twitter/X thread. His approach is "surprisingly vanilla," which is itself the point: the tool is designed so you can use it, customize it, and hack it however you like. The most important takeaway: verification is "probably the most important thing to get great results" and can "2-3x the quality of the final result."

---

## Key Quotes

> "We intentionally build it in a way that you can use it, customize it, and hack it however you like. Each person on the Claude Code team uses it very differently."

> "Probably the most important thing to get great results: verification. It can 2-3x the quality of the final result."

## Key Themes

#claude-code #creator-perspective #workflow #primary-source #verification

### Parallel Sessions at Scale

Cherny runs 5 Claude instances simultaneously in his terminal (numbered tabs 1-5) with system notifications, plus 5-10 additional instances on claude.ai/code and the iOS app. He teleports sessions between platforms and hands off work between local and web environments. This is a much higher degree of parallelism than most users attempt.

### Model Choice

He exclusively uses Opus 4.5 with thinking for everything, reasoning that superior steering and tool-use capabilities make it faster overall than smaller models requiring more correction. Bigger model, fewer retries.

### Institutional Knowledge via CLAUDE.md

Teams maintain CLAUDE.md files checked into git repositories, where the team collectively documents Claude's errors and adds preventive instructions. This is the same pattern described in [[CLAUDE.md (Universal)]] -- the file becomes a shared institutional memory that persists across sessions and team members.

### Plan Then Execute

Sessions typically begin in Plan mode (shift+tab twice), enabling back-and-forth refinement before switching to auto-accept edits mode for implementation. This matches the pattern in [[Trycycle]] and [[How to Write a Good Spec for Agents]] -- invest in planning before turning the agent loose.

### Slash Commands and Subagents

Custom commands stored in `.claude/commands/` automate frequent workflows. Example: `/commit-push-pr` uses inline bash to pre-compute git status, reducing model back-and-forth. Specialized subagents like code-simplifier and verify-app automate post-generation tasks.

### Permission Management

Rather than `--dangerously-skip-permissions`, he pre-allows safe bash commands via `/permissions`, with common configurations shared team-wide in `.claude/settings.json`. This is the sane middle ground between YOLO mode and constant permission prompts.

### MCP Integration

Claude Code integrates with external systems through MCP servers: Slack posting and searching, BigQuery queries for analytics, Sentry error log retrieval, Chrome extension testing for UI verification. This is the tool-use pattern that makes coding agents more than just code generators.

### Verification is the Force Multiplier

Cherny's verification approach: Chrome extension-based browser testing, automated UI interaction, test suite execution, domain-specific validation. For long-running operations, he uses verification agents, Stop hooks, or the ralph-wiggum plugin. The claim that verification 2-3x quality is consistent with [[Feedback Loop is All You Need]] -- the feedback loop, not the generation step, determines output quality.

### Code Quality via Hooks

A PostToolUse hook handles code formatting after generation, addressing the remaining 10% of formatting concerns to prevent CI failures. This connects to [[Pre-Commit Lint Checks]] -- automated enforcement beats instructions.

## Critical Analysis

The value here is provenance. When the inventor of a tool explains their workflow, you're getting design intent, not just one user's adaptation. Several patterns stand out as deliberate design choices rather than emergent workarounds:

1. **Plan mode exists because it works.** The shift+tab workflow isn't accidental -- it reflects the observation that refinement before execution beats correction after.
2. **CLAUDE.md is team memory.** The creator uses it as collective error documentation, which suggests that's its intended purpose, not just a prompt engineering trick.
3. **Verification over generation.** The 2-3x quality claim positions verification as more important than the initial generation -- the agent's first draft is the starting point, not the deliverable.
4. **Parallelism is normal.** Running 5-15 instances simultaneously suggests the tool was designed for this. Compare with [[Addy Osmani's Workflow]] (power user perspective) and [[Claude Code on the Go]] (mobile-first adaptation) for different scaling approaches.

The gap: Cherny doesn't discuss failure modes or situations where Claude Code struggles. The creator's perspective is inherently optimistic. For the complementary view -- where agent-driven development breaks down -- see [[The Mythical Agent-Month]] and [[Slowing the Fuck Down]].

---
*Sources: [[summary/how-boris-uses-claude-code]]*
*Last updated: 2026-05-14*
