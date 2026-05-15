# How to Write a Good Spec for Agents

Addy Osmani's definitive guide to writing specifications that make AI coding agents productive. Five principles: start with high-level vision and let AI draft details, structure like a PRD, break into modular prompts, build in self-checks and domain knowledge, and test/iterate continuously. The spec becomes persistent documentation that grounds the agent across sessions.

---

## Key Quotes

> "Vague prompts mean wrong results."

> "All phases have specific jobs, and you don't move to the next one until the current task is fully validated."

> The "lethal trifecta": speed (fast iteration), non-determinism (variable outputs), and cost (encouraging corner-cutting).

## Key Themes

#specs #methodology #agentic-coding #prds #best-practices

The most actionable contribution is the six essential spec areas identified from analyzing 2,500+ agent configuration files: commands, testing, project structure, code style, git workflow, and boundaries. The three-tier boundary system (always do / ask first / never do) is simple and immediately implementable.

The "curse of instructions" finding -- that model performance drops as instruction count increases -- explains why monolithic prompts fail and modular specs succeed. Separate spec files per domain, pull only relevant context per task, run parallel agents on non-overlapping work.

The distinction between "vibe coding" (rapid exploration) and "AI-assisted engineering" (disciplined, production-ready) is one Osmani draws explicitly. They require fundamentally different specs -- or no spec at all for the former.

Connects to [[Compound Engineering]] (specs as compounding infrastructure), [[Components of a Coding Agent]] (the context management that specs feed into), [[Building 200+ Integrations with OpenCode]] (where skills served as modular specs), and [[claude-code-config (Trail of Bits)]] (which implements many of these recommendations as concrete configuration).

## Critical Analysis

This is the most complete practical guide to spec-writing for agents available. The weakness is that it's optimistic about the spec-writing process itself -- "start with a product brief and let AI expand it" assumes you know what you want, which is often the hard part. The iterative refinement loop (test, discover incompleteness, update spec, re-sync) is realistic but time-consuming, and there's tension between "specs are persistent documentation" and "specs evolve continuously." At what point is the spec being updated so often that it's not a stable reference anymore? Still, for anyone doing serious work with coding agents, this is required reading alongside [[Slowing the Fuck Down]] for the philosophical counterweight.

---
*Sources: [[raw/how-to-write-a-good-spec-for-agents]]*
*Last updated: 2026-05-14*
