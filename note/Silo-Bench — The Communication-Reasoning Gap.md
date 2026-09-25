# Silo-Bench — The Communication-Reasoning Gap

SILO-BENCH (arXiv 2603.01045, Jiang et al., v1 Mar 2026, v2 Apr 2026) is a role-agnostic benchmark of 30 algorithmic tasks across three communication-complexity levels, run over 54 configurations and 1,620 experiments. Its finding is the sharpest empirical statement yet of a suspicion the practitioner literature keeps circling: multi-agent LLM systems are good at *moving* information between agents and bad at *computing with* it. Agents spontaneously form task-appropriate coordination topologies — the social layer works — but the reasoning-integration stage, where distributed state must be synthesized into one correct answer, fails systematically. And coordination overhead compounds with scale until it eliminates parallelization gains entirely.

---

## Key quotes

> "Whether agents can reliably compute with distributed information, rather than merely exchange it, remains an open question."

The paper's framing cuts cleanly: exchange and computation are different capabilities, and most multi-agent demos only demonstrate the first.

> "The failure is localized to the reasoning-integration stage where agents often acquire sufficient information but cannot integrate it."

This is the most useful part — the failure isn't diffuse model incompetence, it's a specific, diagnosable stage. That makes it an engineering target: the fix is in the integration mechanism (shared state, aggregation, a synthesizer role), not in more message-passing.

> "Coordination overhead compounds with scale, eventually eliminating parallelization gains entirely."

> "Naively scaling agent count cannot circumvent context limitations."

The two quotes together are a direct rebuttal of the "just spawn more agents" school: distribution is not a substitute for context, and past some point it's strictly worse.

## Key themes

- #concept — the Communication-Reasoning Gap: topology formation succeeds, synthesis fails
- #concept — coordination overhead as a scaling tax that eats parallelization wins
- #tool — SILO-BENCH itself: 30 tasks, 3 complexity levels, code released as a progress tracker

## Analysis

The paper's sting is in what it does *not* find broken. Agents already self-organize sensible topologies and communicate actively — the social machinery of multi-agent systems works. What fails is the part no one demos: one agent holding the integrated answer. This nuances the optimistic orchestration literature (subagent fan-out, planner/worker trees) by showing the fan-out is the *easy* half; the reduce step is where systems die. It also gives a clean mechanism-level explanation for a pattern practitioners report anecdotally — multi-agent pipelines where every agent behaved reasonably and the process still failed.

Two caveats. First, algorithmic tasks are the worst case for distributed synthesis — they demand exact integration of state, whereas real agentic work (code, review, triage) is more tolerant of partial integration, so the gap may be smaller in practice than in the benchmark. Second, "agents fail to integrate" partially restates the context-limitation problem it claims to transcend: a single-context agent also fails to integrate, just differently. The honest reading is that SILO-BENCH measures where the wall is, not that no architecture can clear it.

The actionable takeaway is architectural: treat integration as a designed component — a deterministic spine or a dedicated synthesizer with full visibility — rather than trusting emergent message-passing to do the reduction.

## Related pages

This strengthens [[Agent Swarm Model Economics]], which documented coordination failure modes at scale in production; SILO-BENCH supplies the controlled-benchmark version of the same phenomenon and shows the overhead is structural, not incidental. It nuances [[Dynamic Workflows in Claude Code]], whose fan-out and tournament patterns assume spawned subagents can aggregate their own results — the gap says the aggregation step needs explicit design. It complicates [[Multi-Agent AI Systems Are Organizations]]: the organizational metaphor implies coordination solves the problem, when the benchmark shows coordination is exactly what works while integration fails. And it echoes [[The Mythical Agent-Month]]'s skepticism about linear scaling of agent capacity with agent count.

---
*Sources: [[raw/2603-01045]], [[summary/2603-01045]]*
*Last updated: 2026-09-25*
