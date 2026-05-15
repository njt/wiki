# Planning With Files

A Claude Code skill implementing persistent markdown planning -- the workflow attributed to Manus AI (acquired by Meta for $2B). The core principle: "Context Window = RAM (volatile, limited); Filesystem = Disk (persistent, unlimited)." Three files track state across sessions: `task_plan.md` (phases and progress), `findings.md` (research and discoveries), `progress.md` (session logs and test results). Hooks force the agent to re-read the plan before decisions and verify completion before stopping.

---

## Key Quotes

> "Context Window = RAM (volatile, limited); Filesystem = Disk (persistent, unlimited)"

## Key Themes

#planning #persistence #context-management #hooks #manus-pattern #file-based-memory

The RAM/disk metaphor is the clearest articulation of why file-based persistence matters for AI agents. The context window is volatile and limited; anything important should be written to disk. The hook-based enforcement (PreToolUse re-reads the plan; completion verification before stopping) is what makes this actually work rather than just being a suggestion.

The evaluation results are striking: 96.7% pass rate with the skill vs. 6.7% without. That's the difference between a structured agent and a wandering one.

## Critical Analysis

This is solving a real and important problem -- context loss across sessions -- with a simple, elegant mechanism. The 21.1k stars suggest it resonates. The three-file structure is opinionated in a useful way: you don't have to decide what to track, just use the template.

The weakness is that file-based planning doesn't scale to projects with deep dependency trees or parallel workstreams. One `task_plan.md` works for linear workflows; for complex projects you'd need something more like a DAG. Compare with [[Claude-Mem]] (database-backed memory with vector search) and [[CodeMira]] (similar but for OpenCode). Planning With Files trades sophistication for simplicity -- and for most single-agent workflows, simplicity wins.

The Manus attribution is interesting context. If Meta paid $2B for the company that popularized this pattern, the pattern clearly has value -- though $2B buys a lot more than a planning skill.

---
*Sources: [[raw/planning-with-files]]*
*Last updated: 2026-05-14*
