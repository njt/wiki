# Online Harvey Agent Platform (Joey Wang)

Joey Wang's talk on Spectre, Harvey's internal agent platform, argues that the hard part of agentic coding is not getting a model to write code but running everything around it: shared secrets and test stacks, a follow-up loop from alert to reviewed PR, programmatically callable actions, and — when hundreds of agents hit the same cluster — alert deduplication and queue-based admission control. It ends where the interesting problem begins: merge contention on a moving main branch.

---

## What it says

Spectre started as a coding-agent platform and escaped: Slack data questions, marketing prototypes, agents started more than once a minute around the clock. Once a harness can reach company tools and do useful work, users bring tasks its builders never anticipated — which is exactly the point, and exactly why a single successful agent run reveals nothing about the failure modes of shared infrastructure.

Three moves:

1. **Own what happens after the code is written.** Don't replace engineers' favourite agents; host them. Supply secrets, a real test stack, alerts, tickets, policies, capacity. Build the follow-up loop — overnight alert → proposed fix → real stack validation → proof on the PR → CI-failure retry — with human approval kept distinct from patch generation and auto-merge. If you build one thing, build that.
2. **Make every action callable, then expect agents to call it.** A CLI over the platform's operations lets laptop agents, sandbox agents, and CI share one execution layer. The corollary: a UI has a human rate limit; a coordinator doesn't. Your API design sets your worst-case load.
3. **Deduplicate, then make capacity fair.** An alert storm spawned ~900 agents chasing one bug. Fix: join repeated alerts to in-progress investigations. Then a *legitimate* engineer's CLI burst overloaded the cluster honestly — requiring admission control: queue for a place vs worker, interactive work reserved, per-person concurrency caps, borrowing when idle. One person, agent, or alert must not own the cluster.

The talk closes honestly on unfinished problems: execution sped up, so pressure moved to merge waits and CI on a moving main branch; unattended automations need budgets, expiry, and failure policies (one disabled after five consecutive failures). Wang even refuses to attribute which of several simultaneously-shipped fixes improved things.

---

## Key quotes

> "The difficult part was not persuading a model to write code. It was keeping everything around those agents working as more people found ways to use them."

The platform thesis in one line — and it matches what the factory literature keeps finding: the model is the easy part.

> "If continuous integration fails, a follow-up loop reads the failure and tries again. By morning, the change is waiting in the same review process as other changes, rather than stranded at the first error."

The follow-up loop as the single most valuable component is a notable judgement call: it's the piece that converts agent output into *reviewable* output.

> "A person clicking through a UI has a natural rate limit. A script or coordinator can create hundreds of tasks before anyone notices the cluster is full."

Every agent-facing interface is also a capacity admission valve whether you designed it as one or not.

> "More agents were not more independent evidence. They were duplicate demand."

The alert-storm lesson in its sharpest form — near-identical to what queueing theory says about correlated arrivals.

---

## Themes

#pattern — the follow-up loop as platform plumbing; human approval distinct from patch generation
#pattern — admission control: queue-for-place vs queue-for-worker, per-principal concurrency caps, borrowing
#concept — duplicate demand: correlated triggers (alerts, bursts) are not independent work
#tool — Spectre as shared harness infrastructure hosting multiple agents

## Opinion

The most refreshing thing here is the honesty about attribution and unfinished work. Wang refuses to credit individual fixes that shipped together, and he leaves the merge-contention problem explicitly open rather than papering over it with a roadmap slide. That matters because the talk's deepest claim — fix one bottleneck and meet the next one — only works if you're actually measuring, not narrating.

The alert-storm story is quietly the most transferable. ~900 agents on one bug is the agentic version of the thundering herd, and the fix (join to in-progress investigations) is classic idempotency/dedup work, not AI work. The industry's reflex is to add more agents for resilience; Spectre's data says that's often just duplicate demand with extra steps. The follow-up story is less glamorous: the "admission control" machinery (queues, caps, borrowing, fairness) is textbook multi-tenant systems engineering. That convergence is the point — agent platforms are not a new discipline, they're platform engineering with worse arrival patterns.

The unresolved ending is the real contribution: when generation is free, merge contention on main becomes the shared resource, and no amount of agent parallelism fixes a coordination problem. Spectre's framing — the platform's value is owning everything between patch and merged change — is a cleaner statement of the control-plane thesis than most.

## Related pages

This talk is the strongest production counterexample to [[The Case Against Building Your Own Agent Platform]] — Harvey built one anyway, and the reason is precisely the company-specific glue (secrets, test stacks, alert topology) a vendor can't own. Its queue-vs-worker admission control nuances [[Queues Don't Fix Overload]]: the problem wasn't that queues fail, but that unbounded *enqueue* upstream of them does — dedup and per-principal caps had to come first. The alert storm and merge-contention bottleneck both feed [[Multi-Agent Systems Have a Distributed Systems Problem]], as concrete industrial instances of correlated arrival and shared-resource contention. And the closing claim that follow-up loops and review integration matter more than raw execution speed strengthens [[Structural Backpressure Beats Smarter Agents]] — back pressure showed up here not as a design principle but as the fire that had to be fought with queues and caps.

---
*Sources: [[raw/online-harvey-agent-platform]], [[summary/online-harvey-agent-platform]]*
*Last updated: 2026-11-26*
