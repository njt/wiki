# agent-pr-replay

Takes merged PRs from any repository, reverse-engineers the task prompt, runs Claude Code against it, and compares what the agent did versus what humans actually shipped. The result is targeted, empirical guidance for improving agent performance in a specific codebase.

---

## Key Quotes

> "The best way to improve an agent's ability to work in a codebase is to observe its default behavior, measure the gap against real human solutions, and steer it based on evidence."

## Key Themes

#tool #agent-evaluation #benchmarking #empirical #agentic-coding

The methodology is simple and powerful: use real merged PRs as ground truth, replay the scenario with an agent, diff the results. This generates CLAUDE.md steering rules, reusable skills, and statistics about tool usage and file access patterns -- all grounded in evidence rather than theory.

The PyTorch example findings ("prefer deletion over defensive programming," "minimize changes to existing code structure") are the kind of codebase-specific guidance that generic best practices can't provide. Every codebase has its own idioms, and the only way to discover them is to observe what the agent gets wrong.

At ~$4 per analysis session, this is cheap enough to run regularly. The cost of one bad PR is far higher than the cost of understanding your agent's blind spots.

## Critical Analysis

This is the empirical approach to agent improvement that [[Feedback Loop is All You Need]] argues for. Instead of writing CLAUDE.md rules from intuition, you generate them from data. Instead of hoping the agent follows your conventions, you measure where it diverges and encode corrections.

The limitation is that it only works for codebases with a history of good PRs. If your human PRs are sloppy, replaying them against an agent doesn't tell you much useful. The ground truth is only as good as your team.

The security caveat (runs in git worktrees, uses Claude API) means this should run in trusted repos or sandboxed environments. At $4/session, runaway cost is a concern if you automate it against hundreds of PRs.

See also [[session-analysis]] for understanding the dynamics of individual sessions, [[claude-replay]] for visualizing what happened, and [[Optimise Anything]] for automated optimization of agent skills.

---
*Sources: [[raw/agent-pr-replay]]*
*Last updated: 2026-05-14*
