# Orchestrator - Worker Skill

A single Claude Code skill that combines orchestrator and worker roles rather than separating them into two files. Part of the n-skills collection ("Curated plugin marketplace for AI agents -- works with Claude Code, Codex, and openskills"). The interesting design choice is co-locating coordination logic and execution logic in one place.

---

## Key Quotes

> No quotes available (URL returned 404).

## Key Themes

#orchestration #multi-agent #skills #design-patterns

The single-skill-not-two decision is the interesting bit. Most orchestration approaches (including [[Awesome Agentic Patterns]]' 51 orchestration patterns) assume a separation between the orchestrator and the workers. Merging them reduces the overhead of multi-skill coordination but potentially blurs separation of concerns. For simple workflows, one skill is less fiddly. For complex ones, the merged approach might become a tangle.

## Critical Analysis

Without the actual SKILL.md content (404), it's hard to evaluate the execution. The concept is worth noting as a design alternative: not everything needs the overhead of a multi-agent architecture. Sometimes "one agent that knows how to break work into pieces and do those pieces" is simpler than "one agent that coordinates other agents." The question is where the complexity threshold lies. See [[Planning With Files]] for another take on structured task execution within a single agent context, and [[Awesome Agentic Patterns]] for the full orchestration pattern space.

---
*Sources: [[raw/orchestrator-worker-skill]]*
*Last updated: 2026-05-14*
