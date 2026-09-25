# Multi-Agent Systems Have a Distributed Systems Problem

Christopher Meiklejohn — a decade-deep CRDT and distributed-systems researcher (Lasp, Partisan, Filibuster, Basho) — argues that multi-agent LLM systems reproduce, by construction rather than by accident, every classic failure category of distributed systems: lost updates, stale reads, crash recovery, causal ordering, partitions, and Byzantine faults. His evidence is hands-on: two Claude Code instances in different worktrees both wrote migration 267 and one silently clobbered the other.

---

## The Argument

The essay opens with a story that is also a diagnosis. Two agents, two worktrees, one migration filename, different schemas, silent overwrite. Meiklejohn's reaction — laughter — is the tell of someone recognising an old enemy in new clothes: a lost update, the problem CRDTs were invented for, playing out in a directory of SQL files.

His survey of the multi-agent literature lands the punch:

> Shared mutable state with no formal concurrency control. No fault model. No reasoning about what happens when agents disagree.

ChatDev has "communicative dehallucination" (a role-reversal clarification heuristic) but no causal ordering across chat chains. MetaGPT's shared message pool is real progress — publish-subscribe with role-profile subscriptions — but pub-sub is not concurrency control: it tells you what other agents produced, not whether you're reading a stale version. AutoGen has no persistent shared state at all. Three of the field's flagship frameworks, one shared blind spot.

The centrepiece is a five-item taxonomy of inevitable failures:

- **Conflicts and stale reads** — two agents edit the same file; one's work vanishes. Or an agent picks a bug from the tracker that was fixed ten minutes ago.
- **Failure and recovery** — the crash-recovery model, but the replicas hit context limits and hallucinate instead of merely segfaulting.
- **Ordering without a clock** — a triage agent assigns a bug to A while B, who saw the report first, is already fixing it. Lamport 1978 and vector clocks solve this; "the multi-agent world hasn't noticed yet."
- **Partition tolerance** — API timeouts and rate limits split agents into diverging groups whose states must merge losslessly on reconnect.
- **Byzantine faults** — an agent hallucinates a plausible fix, passes its own tests, ships it; downstream agents build on top. "Every agent is a potential Byzantine actor every time it responds."

The honest coda matters: he doesn't claim the distributed-systems toolkit transfers cleanly. LLM agents aren't database replicas — they hallucinate, lose context, and make confident decisions on incomplete information. Whether CRDTs, version vectors, and fault injection carry over is, to him, "the most interesting question I've encountered in a long time."

## Key Themes

#concept #pattern — coordination as the missing layer beneath agent frameworks

## Analysis

The strongest thing about this essay is provenance: most "multi-agent systems are just distributed systems" takes come from people who arrived at agents first. Meiklejohn arrived from the other direction — Riak, SyncFree, a 1,000-node CRDT deployment, a 10-year influence award — and his authority is earned, not asserted. That makes the essay's restraint credible: he is not saying "use CRDTs," he is saying the *problem shape* is identical and the solution space is unexplored.

Two complications worth pushing on. First, the Byzantine framing is doing heavy lifting: distributed systems handle Byzantine faults with quorum thresholds (3f+1 replicas), but an agent's hallucinations aren't adversarial, they're stochastic — more like soft errors than traitors, and possibly better served by the verification-heavy patterns the agent world has already developed than by Byzantine consensus. Second, and related: the essay's own caveat undercuts the taxonomy slightly. If agents differ from replicas in the exact dimensions where the hardest distributed-systems techniques bite (Byzantine behaviour, partial observability), then the transferable fraction may be the 1970s–80s core (ordering, versioning, recovery) rather than the hardest problems. That's still a big deal — stale-read detection alone would have saved migration 267.

It's also a quiet indictment of the field's literature: three well-cited frameworks, and the coordination layer underneath is message passing. Fifty years of research, none of it imported.

## Relation to the Wiki

This source sharpens [[Agent Swarm Model Economics]], which catalogued five coordination failure modes at scale empirically — Meiklejohn supplies the formal vocabulary (lost updates, causal ordering, Byzantine actors) that explains *why* Cursor hit those failures. It complicates [[Deterministic Spine, Agentic Leaves]]: that piece prescribes deterministic components owning state as the fix, while Meiklejohn argues the deeper gap is formal concurrency control on shared state — compatible, but his framing is more foundational. It also gives [[Stash — Conflict-Free Folder Sync]] an unexpected dignification: a practical CRDT-adjacent merge engine for exactly the kind of shared-directory state (agent memory, skills files) that produces Meiklejohn's lost updates.

---
*Sources: [[raw/multi-agent-systems-have-a-distributed-systems-problem-html]], [[summary/multi-agent-systems-have-a-distributed-systems-problem-html]]*
*Last updated: 2026-09-25*
