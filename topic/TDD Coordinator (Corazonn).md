# TDD Coordinator (Corazonn)

Schuyler's `/go` slash command for Claude Code: a TDD coordinator that orchestrates subagents through red/green/refactor cycles, with a mandatory "Rule of Two" quality gate where every work product must be reviewed by a different agent. The most rigorously phase-gated agent orchestration command I've seen — five phases, 15+ steps, Marx Brothers role names, and a hard rule against the coordinator ever writing code itself.

---

## Key Quotes

> "You are the TDD Coordinator. Your role is to orchestrate subagents through a complete test-driven development workflow."

The coordinator-as-conductor model. The opening instruction ("Yallah!") sets the tone — this is a ritual, not just a tool.

> "Every work product must be reviewed by a different agent than the one that produced it."

The **Rule of Two**. This is the core quality mechanism. A primary agent produces, a different agent evaluates against Quality, Correctness, and Adherence. Critical issues block progress. This is [[Compound Engineering]] made operational: don't manually review when you can build a system that does it.

> "If stuck after 4-5 attempts, ask user for help."

The no-thrashing escape hatch. Most orchestration systems optimistically loop forever. This one has a hard cutoff and escalates to the human. That's honest engineering.

> "Karl writes failing tests first (red); Zeppo gates the tests; Karl implements; Zeppo gates the implementation — loop until approval."

The red/green/refactor heartbeat, but with an independent gate at every step. This is TDD where the test-writer and code-writer are prevented from colluding. Compare with [[Trycycle]]'s fresh-agent-at-every-review-stage pattern — same insight, different implementation.

---

## Key Themes

#agent-orchestration #tdd #quality-gate #claude-code #slash-command #multi-agent

The five-phase structure (Understand → Implement → Review → Document → Verify) echoes [[recursive-mode]]'s file-backed phase gates, but with role specialization instead of artifact numbering. The key difference: recursive-mode is single-agent with audit loops; this is multi-agent with cross-review gates.

---

## Critical Analysis

**The Rule of Two is the real innovation here.** Everyone doing multi-agent orchestration has some version of "review your work," but Schuyler formalizes it into a hard constraint: different agent, three explicit criteria, critical-vs-minor severity classification. This is what separates a suggestion from a gate. The Marx Brothers naming (Groucho analyzes, Karl codes, Zeppo gates, Harpo documents) isn't whimsy — it makes the roles memorable enough that you can reason about the pipeline without looking at a diagram.

**The weakness is the single-model problem.** All five Marx Brothers run on the same underlying model. [[Fresh Eyes]] has the sharper insight: cross-model review catches blind spots that same-model review misses. A version of this with Karl on Claude and Zeppo on Codex would be strictly stronger. Schuyler knows this — the coordinator role itself acts as a meta-gate — but the individual agent reviews are all same-model.

**Five phases is a lot of handoffs.** Each handoff is a context reset, and context resets are where agents lose the thread. The coordinator's TodoWrite tracking mitigates this, but there's a real throughput cost. For a bug fix that takes one agent two turns, this would burn 10+ turns. The command is optimized for correctness over speed, which is the right tradeoff for critical-path code, but overkill for CSS tweaks.

**This belongs in the planner/worker/judge taxonomy.** The coordinator is the planner, Karl and Harpo are workers, and Chico and Zeppo are judges. Cursor arrived at this pattern independently after flat self-coordination failed ([[Scaling Long-Running Agents]]). Schuyler is implementing it at the slash-command level rather than the platform level, which means it works *today* in vanilla Claude Code.

---

## Related Pages

- [[Agent Orchestration]] — synthesis of multi-agent coordination patterns
- [[Trycycle]] — fresh-agent review at every stage; same quality-gate philosophy
- [[Fresh Eyes]] — cross-model review addresses a blind spot this doesn't
- [[Compound Engineering]] — "add a system, not manual review" — this IS compound engineering
- [[Scaling Long-Running Agents]] — Cursor's planner/worker/judge finding, which this implements
- [[recursive-mode]] — similar phase-gated workflow, single-agent approach
- [[Orchestrator - Worker Skill]] — similar orchestrator/worker separation
- [[Spec-Driven Development]] — the specs/tests/code triangle this coordinator enforces
- [[Agent Coding Workflow]] — the daily practice this command structures

---
*Sources: [[summary/corazonn-go]]*
*Last updated: 2026-05-14*
