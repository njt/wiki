# Multi-Agent Orchestration Across Repositories

Adam Bertram's Telerik essay separates the two coordination problems hiding inside "orchestration": coordinating agents working on separate tasks, and coordinating changes that span repositories with different interfaces, dependencies, and delivery gates. Solving one does not solve the other — and his opening scenario (11 green PRs across five repos, broken auth service) shows both failing at once because nobody owned the part *between* the agents.

---

The essay's central image is worth dwelling on: every test suite is green and the system is broken. Each agent did its assigned job; no one saw that Agent A's change touched code Agent B depended on. This is the software-delivery version of the classic multi-agent failure — per-agent correctness without global coherence.

## Key quotes

> An agent, like a human, knows only the context it is given; without context it flies blind.

The framing device for the whole piece: orchestration is context logistics. Everything that follows — handoffs, durable plans, cross-repo visibility — is about making sure the right context survives a session boundary.

> There are two coordination problems here. Multi-**agent** orchestration coordinates multiple agents working on separate tasks. Multi-**repository** orchestration coordinates changes that span codebases with different interfaces, dependencies and delivery gates.

The genuinely useful conceptual contribution. Most orchestration writing conflates them; Bertram's point that repo heterogeneity is its own problem (one repo's gate is a merge queue, another's is a named reviewer group, a third's is a nightly build and a person who says yes) is a sharper taxonomy than most.

> Orchestration should make agents inherit those controls rather than create separate ones.

Against the grain of agent-harness enthusiasm: your CI/CD system already encodes years of parallel-work discipline designed for humans. Build agent plumbing onto it, don't fork it. Getting this wrong shows up in delivery numbers — he cites DORA's 2025 finding that AI adoption still correlates with rising delivery instability.

> Only OS permissions or sandboxing draw a line an agent instruction cannot cross.

On Git worktrees as the isolation primitive: each agent gets its own working directory, index, and HEAD. But he's honest about the limit — an agent told to stay in its tree can still wander. Isolation is a property of the filesystem, not the prompt.

> Research on multi-agent system failures has found that many failures come from this kind of misalignment and from the way the workflow is designed, not necessarily from the capabilities of the model.

The pivot from tooling to process design: treat the handoff as a versioned, reviewable artifact carrying four fields — **scope** (which paths may be touched), **interface requirement** (the contract the work must satisfy), **base** (the commit the worktree started from, so a stale plan is detectable), and **evidence** (commands and tests already run). This is interface-contract thinking applied between agents.

> Larger context windows do not remove the need for this coordination.

A timely rebuke to the "just give one agent everything" school: a single worker holding the whole change makes a bad part harder to isolate or reject. Coordination is not a workaround for small contexts; it's a property of good decomposition.

> The prerequisite: a delivery workflow explicit enough to hand to something that will not ask you what you meant.

The best line in the piece, and the honest diagnosis: most multi-agent failures are symptoms of workflows that only worked because humans silently filled in the gaps.

## Themes

#concept #pattern #orchestration #workflow

## Analysis

The essay is strongest where it's concrete — the four-field handoff artifact, worktrees-from-fresh-main, hotspot-file locking, the integration-run-as-merge-prerequisite pattern. These are real, actionable patterns, and the "isolation precedes parallelism" ordering is stated better than in most longer treatments.

It's weakest where it's marketing-shaped. The closing plug for Progress Forge (formerly Progress Agent Harness) is hard to ignore: the essay reads as content marketing for a commercial orchestration product, and its most rhetorical moves — "composition has an order," the three-role control layer "one framework you can use" — gesture at a framework that is never actually specified. The reader gets a checklist of problems and a product, not a design.

The human-review bottleneck gets one paragraph and deserves more: "Ten agents feeding one reviewer is a queue. Orchestration can route and summarize that queue, but it can't replace the judgment required to work through it." That's a caveat acknowledged, not a problem solved — and it's the problem that actually kills these systems at scale.

Still, the two-problems taxonomy and the four-field handoff are genuinely additive to the orchestration literature, and the "write it down; agents won't ask what you meant" closing is the correct prerequisite, stated memorably.

## Related pages

This source strengthens [[Handing the Agent the Whole Job]] — the same author's earlier Telerik essay — by applying its "write the finish line before the first run" doctrine to the hardest case, where one workflow spans five repos and three different merge gates. It nuances [[Multi-Agent Systems Have a Distributed Systems Problem]]: Meiklejohn frames multi-agent coordination as missing concurrency control, while Bertram shows the practical mitigation layers (worktree isolation, hotspot locking, integration runs) that practitioners reach for instead. It complicates [[Silo-Bench — The Communication-Reasoning Gap]] by supplying an engineering answer to the benchmark's diagnosis — the four-field handoff artifact is precisely a fix for the information-integration failures Silo-Bench isolates. And it extends [[Loop Engineering]]'s worktrees-and-state taxonomy upward from the single-repo loop to the cross-repository composition layer, where per-branch CI can't see the dependency order.

---
*Sources: [[raw/multi-agent-orchestration-software-delivery-patterns-multi-repository-workflows]], [[summary/multi-agent-orchestration-software-delivery-patterns-multi-repository-workflows]]*
*Last updated: 2026-09-29*
