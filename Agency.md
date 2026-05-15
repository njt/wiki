# Agency

A self-hosted engine for composing AI agents from reusable natural-language primitives (roles, desired outcomes, trade-off configurations). Instead of rewriting monolithic agent prompts when something underperforms, you modify individual building blocks. Runs as a background MCP service that Claude Code talks to.

---

## Key Themes

#agent-composition #modularity #mcp #prompt-engineering #reusability

The core insight is treating agent prompts like software components: decompose them into small, testable, recombinable pieces. Role components define capabilities (50-150 chars), desired outcomes define success criteria, trade-off configurations handle conflicting values. The system selects and composes these into specialized agents per task.

This is the prompt engineering equivalent of the Unix philosophy -- small pieces loosely joined. It connects to [[Elements of Agentic Systems Design]] (particularly the Context and Reasoning elements) and sits in tension with [[What I learned building an opinionated and minimal coding agent]] where Zechner argues that minimal prompts work fine because frontier models already understand coding agents through RL training.

## Critical Analysis

Strong: the modularity thesis is sound. Monolithic prompts are fragile, and the idea of accumulating performance data to learn which primitive combinations work is a step toward systematic prompt optimization.

Weak: the Elastic License 2.0 prohibits commercial hosting, which limits adoption for the most natural deployment pattern (a shared team service). The "composes small agents for discrete tasks; not for long-running autonomous systems" scope limitation is honest but raises the question of whether the overhead of composing primitives is worth it for small tasks. If your agent is doing one thing, a well-crafted prompt might just be simpler.

The real test is whether the primitive library grows organically or stagnates. Reusable components only work if the community contributes and curates.

---
*Sources: [[raw/agency]]*
*Last updated: 2026-05-14*
