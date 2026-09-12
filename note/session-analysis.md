# session-analysis

Leonard Lin's tools for understanding how long things actually take when working with AI coding assistants. Built during FSR4 RDNA3 kernel optimization work, these scripts analyze Claude Code and Codex CLI session JSONL files to extract wall time, active compute time, token consumption patterns, and human-AI collaboration dynamics.

---

## Key Quotes

> "When using AI coding assistants for non-trivial work (like our FSR4 kernel optimization campaign), it's useful to understand how long things actually took."

> "Claude Code cache tokens dominate total API token counts...a session showing 4M total API tokens may have only 6K regular input + 13K output."

## Key Themes

#agent-observability #metrics #token-economics #session-analysis

The cache token insight is the headline finding. If you're looking at your Claude Code bill and seeing 4M tokens, you might panic. But if 3.99M of those are cache tokens (which are much cheaper), the actual cost picture is radically different. This kind of analysis turns "AI is expensive" from a vague complaint into a measurable, optimizable problem.

The wall-time vs. active-compute-time distinction matters for planning. If a session takes 2 hours wall time but the agent was actively computing for 20 minutes, that tells you something about the human bottleneck (reviewing, deciding, prompting) vs. the machine bottleneck. It's the kind of data you need to actually improve your workflow rather than just feeling productive.

## Critical Analysis

This is a research tool, not a product. It requires manual JSON piping and filtering. But the questions it answers are exactly right. Someone should build this into a proper dashboard -- [[claude-replay]] does the visual replay side, but there's no good tool for the quantitative "how did my agent sessions go this week" view.

The focus on reproducibility planning is forward-looking. If you know how long a particular kind of task takes with an agent, you can plan sprints and estimate costs. Without this data, agent-assisted development is a gamble.

See also [[claude-replay]] for the visual replay angle, [[agent-pr-replay]] for measuring agent vs. human performance on the same tasks, and [[Feedback Loop is All You Need]] for why you need observability in the agent loop.

---
*Sources: [[summary/session-analysis]]*
*Last updated: 2026-05-14*
