# Evolving Agent Teams (arXiv)

An arXiv paper proposing a multi-agent framework that beats single coding agents on open-ended optimization — most strikingly by generating a CUDA kernel that beats the production FlashInfer MLA decode kernel by 1.2x mean speedup, from scratch. The mechanism is not a smarter model but a smarter *organization*: a supervisor that diagnoses when the team has hit a local optimum and restructures the team, backed by a persistent file system of accumulated experience.

---

The paper's diagnosis is familiar to anyone who has run a coding agent on a hard problem for a long time: agents **converge to local optima** on real-world, expert-written code. Given a FlashInfer kernel, a strong single agent finds the obvious wins, plateaus, and then keeps polishing the same plateau.

Evolving Agent Teams' answer has two parts:

1. **A central supervisor that diagnoses and restructures.** When the team stalls, the supervisor doesn't just say "try harder" — it changes the team itself: which agents exist, what they focus on, how they're wired together. This is restructuring as an explicit, first-class control action, which is rarer than it sounds; most multi-agent setups fix the topology at design time.
2. **A persistent file system as the ground truth of experience.** A learning document, an evolution history, and per-team artifacts. Restructuring decisions are grounded in what has already been tried, so the team doesn't cyclically re-explore dead ends. The memory is deliberately dumb — files, not a vector database.

The results are the strongest part of the claim: under a **matched token budget** it consistently outperforms strong coding-agent baselines, which controls for the obvious objection that it's just buying more compute. With an extended budget, the FlashInfer result (1.2x mean speedup over a production kernel, winning 36 of 47 FlashInfer-Bench workloads) plus KDA prefill wins on AMD MI350X (3.01x over vLLM) and B200 (3.44x over FlashKDA), MoE serving on Apple silicon, and a RoboDojo leaderboard win suggest the approach generalizes beyond one benchmark.

## Key quotes

> "existing coding agents converge to local optima on real-world, expert-written kernels"

The framing matters: the failure isn't capability, it's *dynamics*. The agent's loop is stable but stuck. That reframes the supervisor's job as an escape operator, not a quality checker.

> "a central supervisor that diagnoses each local optimum and restructures a team of agents"

This is the interesting design commitment. Multi-agent work usually asks "how many workers, what topology?" and freezes the answer. Here the topology itself is under control of the supervisor and changes over the run — the team is a mutable artifact, like the code.

> "a persistent file system that comprises a learning document, an evolution history, and per-team artifacts, grounding restructuring with accumulated experience"

Filesystem-as-memory again — the same primitive that keeps winning in practice. Notably, the *history of team structures* is itself persisted, so evolution is cumulative across restructurings rather than a fresh restart each time.

## Key themes

- #concept — local optima as the characteristic failure mode of long agent runs, not lack of intelligence
- #pattern — supervisor-triggered team restructuring: topology as a mutable, diagnosed artifact
- #tool — plain files (learning doc, evolution history, per-team artifacts) as durable cross-agent memory
- #concept — matched-token-budget evaluation as the honest way to compare multi-agent vs single-agent

## Analysis

This paper sits squarely in the 2026 consensus that orchestration, not raw model quality, is where headroom lives — and it pushes that argument further than most by making the *organization itself* the thing that evolves. The matched-token-budget comparison is genuinely good methodology; too many multi-agent papers win by spending more tokens and calling it architecture.

The honest caveat is that the headline results come "with an extended token budget." Beating a production kernel by writing a new one from scratch with effectively unlimited compute is impressive but says less about day-to-day practice, where budgets are the constraint. The matched-budget result against *baselines* is the one that generalizes.

There's also a quiet tension the paper doesn't fully resolve: a supervisor that diagnoses local optima and restructures the team is itself an agent making judgment calls about why the team is stuck. What happens when the diagnosis is wrong? The evolution history mitigates repeated mistakes, but the paper's evidence for the *supervisor's* decisions being good — as opposed to the aggregate search being good — is thin. It's a step beyond fixed topologies, not yet a science of when to restructure.

The domain choice is smart, though: GPU kernels have hard, measurable oracles (does it run, how fast), so "escaped the local optimum" is unambiguous. That makes this one of the cleaner demonstrations that agent-team *dynamics* — not just agent-team *existence* — is a lever.

## Connections

- [[Fleet Supervisor (sermakarevich)]] treats the supervisor as a static role handing out work; this paper strengthens that pattern but goes further, making the supervisor's job *restructuring the team itself* when diagnosis says the topology is the problem.
- [[Multi-Agent Orchestration Across Repositories]] covers delegation across codebases at the workflow level; Evolving Agent Teams complicates it by showing the topology should be dynamic and diagnosed, not fixed per-repo upfront.
- [[Compound Engineering Plugin]] also persists a durable learning document that each run reads as grounding; this paper independently converges on the same file-based memory primitive, but applies it to team structure rather than engineering solutions.
- [[Sixteen Agents and a Byte-Exact Oracle]] is the parallel demonstration that hard oracles unlock aggressive multi-agent search; this paper extends that from verification-driven search to *structural* evolution of the search team.

---
*Sources: [[raw/2610-36848]], [[summary/2610-36848]]*
*Last updated: 2026-11-09*
