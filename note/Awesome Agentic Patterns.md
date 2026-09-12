# Awesome Agentic Patterns

A catalogue of 169+ production-ready patterns for AI agents, organized across eight domains. Grew out of Sourcegraph's experience building coding agents. The premise: "Tutorials show toy demos. Real products hide the messy bits." Each pattern must be repeatable, agent-centric, and traceable to public references.

---

## Key Quotes

> "Tutorials show toy demos. Real products hide the messy bits."

## Key Themes

#agentic-patterns #catalogue #orchestration #memory #feedback-loops #security #reliability

The sheer scope is the standout: 51 orchestration patterns, 25 tool-use patterns, 20 context/memory patterns, 21 reliability patterns. This is the reference manual for building agent systems. The categories themselves tell a story -- orchestration and tool use dominate, suggesting that coordination and interface design are the hardest problems, not the AI itself.

The `llms.txt` file for AI assistants to self-discover patterns is a nice meta-touch. The Pattern Explorer, Compare Tool, and Graph Visualization make this more than just a list.

## Critical Analysis

The strength is breadth; the weakness is depth. 169 patterns catalogued means each one gets relatively shallow treatment. The "traceable" requirement (public references required) biases toward patterns that companies have blogged about, not necessarily the most effective ones. The best patterns may be the ones nobody's published yet.

The Sourcegraph provenance is both a strength (real production experience) and a limitation (coding agent focus, not general-purpose agents). Still, this is the best single resource for someone building agent infrastructure. Compare with [[Planning With Files]] (one specific pattern implemented in depth), [[Orchestrator - Worker Skill]] (another specific pattern), and [[Claude-Mem]] (implementing the memory patterns). The catalogue is the map; those projects are the territory.

---
*Sources: [[summary/awesome-agentic-patterns]]*
*Last updated: 2026-05-14*
