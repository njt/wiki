# We Built a Scalable Agent Sandbox

A CloudSquid field report on the three failed approaches — and one that worked — for running hundreds of agents over the same enterprise data. The team's problem is data-plane isolation, not OS-level sandboxing: how do you let 100 agents reconcile invoices, delivery notes, ERP exports, and PDFs in parallel without them overwriting each other? The answer they converged on is a permissioned virtual file system where each agent task gets an isolated copy of only the files it may touch, and changes sync back to the source in a controlled, ordered, auditable way. The sharpest claim is negative: sequential hand-offs between agents are almost never necessary if the workspace is configured correctly upfront.

---

## Key Quotes

> "When you run 100 agents parallely across the same enterprise data, things start to get complex. There are structural limits one hits when you keep adding more agent-driven workflows."

The premise, in plain terms. This is the same threshold [[Agent Swarm Model Economics]] crossed — the moment the coordination cost of shared mutable state stops being an edge case and becomes the architecture.

> "Imagine 20 people editing the same Excel file at once. It crashes or produces conflicting versions. Changes made by people silently disappear, and nobody notices. This just happens when concurrent actors touch a shared resource with no coordination model."

The concurrency analogy that anchors the piece. "Now put agents in those 20 seats instead of people," it continues — they move faster and don't pause to ask. The insight is that agents don't just reproduce the human concurrency problem, they accelerate it.

> "Agents overwriting each other's work, producing conflicting results and even starting a turf war. At scale, this happens frequently, unless the architecture is explicitly built to prevent it."

The citation is loose (a "recent multi-agent research study at Anthropic," unlinked), but the phenomenon is real and matches the five failure modes in [[Agent Swarm Model Economics]] — split-brain design and merge conflicts are the same disease, named differently.

> "Each of the 100 agents in a reconciliation task works inside its own clean environment. It reads what it's permitted to read, writes what it's permitted to write, and cannot see what's been hidden from it. When it finishes, its changes sync back to the true source in a controlled, ordered way with no race conditions and no silent overwrites, fully auditable."

The virtual-file-system design in one paragraph. Notice what's doing the work here: the **permissioned copy**, not the sync protocol. Collision is eliminated at the design boundary by making the shared resource non-shared.

> "We successfully decoupled agent count from collision risk. A hundred agents no longer means a hundred chances to collide, because collision was never possible in the first place."

The thesis in a sentence. This is a [[State-Oriented Consistency]] move applied to files: don't coordinate access to a shared resource, ask each task what it actually needs and give it only that.

> "Sequential hand-offs between agents, the instinct that 'Agent A must finish before Agent B starts', turn out to be almost never necessary."

The boldest claim in the piece, and the least substantiated. It argues coordination overhead is a *design-time* property — eliminated by "Standard Operation Procedures" configured upfront — rather than a runtime choreography problem. That's the [[Specifications as the Product]] argument restated for orchestration: get the workspace/spec right and the coordination collapses to zero.

## Key Themes

#tool #pattern #concept

**The three-approach arc as a cautionary tale about constraint.** v1 was a state machine — a graph of every possible user action with the agent picking from a small pre-defined response set. It failed on four counts the author lists: double build (once as UI for humans, once as a rigid graph for the agent), un-anticipatable edge cases, unused model capability ("the ability to reason broadly... was locked away behind a tightly constrained box"), and — most telling — **model improvements couldn't express themselves**. This is the overlooked cost of deterministic control: a tightly constrained environment freezes out the very capability gains that make the next model worth adopting. It complicates the "linters beat prompts" thesis of [[Guardrails and Feedback Loops]] — deterministic enforcement is good, but over-constraint has a real price.

**Platform ceilings become scale ceilings.** The v3 sandboxed terminal hit Anthropic's hosted-environment file-count cap — a limit that "exists to avoid getting abused by users as free storage or an attack surface." The observation worth keeping: abuse-prevention limits on shared platforms ([[Best Infrastructure Platforms for Coding Agents in 2026]]) are *proportional to nothing about the legitimate workload*, so an enterprise workload can hit them while behaving perfectly well. Legitimate scale and abuse look identical to a hard cap.

**The VFS is copy-on-write with a permissioning twist.** One isolated Google Cloud file store; per task, an isolated copy of only the permitted files. This is the filesystem analogue of [[Building Agents That Don't Break Themselves]]'s copy-on-write checkpointing, but aimed at the *data* plane rather than the execution plane — and it directly answers that page's open question about distributed state (the undo button that's "illusory for distributed state") by making the sync-back, not the write, the governed step.

## Critical Analysis

**What's genuinely valuable here.** The honest three-failures-then-a-win structure is more instructive than a polished architecture post. The state-machine failure in particular — unused model capability, model improvements that can't express themselves — is a real and under-discussed cost of the "tightly controlled environment" school of agent design. And "decoupled agent count from collision risk" is a useful reframe: the alternative to coordination is *not sharing*, which most agent-orchestration discussions skip past in favor of ever-more-elaborate locking and hand-off protocols.

**Where it's thin.** This is a LinkedIn-funnel post ("connect with me on LinkedIn"), not documentation. No author name, no date, no code, no metrics. The critical claim — "collision was never possible" — is only true if the permissioning partition is genuinely disjoint, which reconciliation-shaped tasks (each agent owns its row) make plausible but which the article never generalizes from. What happens when two agents *legitimately* need to write the same field? The "controlled, ordered way" of sync-back is doing enormous work in five words. [[State-Oriented Consistency]]'s framework — ask each piece of state what it needs — is the missing chapter; the article stops at "permissioned copies" without asking when permissioning can't be made disjoint.

**The "sequential hand-offs almost never necessary" claim is the most important and the least defended.** If true, it inverts a lot of [[Agent Orchestration]]'s choreography machinery — the planner/worker hand-offs, the DAGs, the runtime sequencing. But "configured correctly upfront with Standard Operation Procedures" is the same move as "write a better spec": correct, and also where all the actual difficulty lives. The article names the escape hatch without opening it.

**The unverifiable Anthropic reference is worth flagging, not because it's wrong, but because it's doing rhetorical work.** "Agents... even starting a turf war" is evocative and probably true, but citing an unlinked "recent multi-agent research study" lets the strongest empirical claim ride on a source the reader can't check. Compare with [[Agent Swarm Model Economics]], which published the actual failure numbers (68K commits of thrash, 70K merge conflicts). The CloudSquid post is the field-report version of the same finding, and it reads better than it verifies.

**Net.** A useful, readable case study of the data-plane side of agent sandboxing — the side [[A Deep Dive on Agent Sandboxes]] explicitly leaves unaddressed when it asks "what if Agent A needs to share a result with Agent B?" The answer CloudSquid gives is: don't have them share at all; give each a permissioned copy and govern the merge. That's a real architectural position, and worth holding against the two VFS poles — Mirage's one-unified-tree ([[Mirage (VFS)]]) vs. this one's per-task-isolated-copies — which make opposite bets about what "a filesystem for agents" should mean.

---

## Related

- [[A Deep Dive on Agent Sandboxes]] — the OS-level (execution-plane) sandboxing companion; this page fills its open question about inter-agent data sharing
- [[Mirage (VFS)]] — the contrasting VFS bet: one unified POSIX tree over 27 services, vs. per-task permissioned copies
- [[Building Agents That Don't Break Themselves]] — copy-on-write checkpointing on the execution plane; CloudSquid applies the same reflex to the data plane
- [[Agent Swarm Model Economics]] — the same shared-mutable-state collision problem, with the hard failure numbers this post lacks
- [[State-Oriented Consistency]] — the design framework this post is implicitly using ("collision was never possible" = isolate the state, don't coordinate it)
- [[Agent Orchestration]] — the hub this post's "hand-offs almost never necessary" claim pushes against
- [[Security and Sandboxing]] — governance + access controls at scale, which the post keeps as its north star

---

*Sources: [[raw/we-built-a-scalable-agent-sandbox]], [[summary/we-built-a-scalable-agent-sandbox]]*
*Last updated: 2026-08-25*
