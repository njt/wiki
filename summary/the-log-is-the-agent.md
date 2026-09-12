---
url: https://arxiv.org/html/2605.21997v1
title: "The Log is the Agent: Event-Sourced Reactive Graphs for Auditable, Forkable Agentic Systems"
author: Yohei Nakajima
date_fetched: 2026-07-05
date_published: 2026-05-21
topics:
  - agent-architecture
---

# The Log is the Agent: Event-Sourced Reactive Graphs for Auditable, Forkable Agentic Systems

**Author:** Yohei Nakajima — Untapped Capital, activegraph.ai
**arXiv:** 2605.21997v1, submitted May 21, 2026, category cs.AI
**License:** arXiv.org perpetual non-exclusive license
**Open Source:** Apache-2.0 at github.com/yoheinakajima/activegraph

## Abstract

Most agent frameworks organize themselves *around* the language model — conversation loop first, then tools, then rules, logging added later. ActiveGraph inverts this: the append-only event log is the source of truth, the working graph is a deterministic projection of that log, and behaviors react to graph changes by emitting new events. This yields three properties: deterministic replay, cheap forking, and end-to-end lineage.

## 1. Introduction

The conventional pattern "grows by accretion" — chat loop, tools, rules, logging bolted on after the fact, memory stored as compressed summaries or embeddings. The log is a "byproduct: an audit artifact written alongside the real computation, never the substrate of it."

ActiveGraph asks an inverse question: what if the log *were* the agent — "the rules an agent follows (and the changes to those rules), the tools it is given and the calls it makes, and the content it produces were all the same kind of thing — events in a single append-only log?"

The system is a "direct descendant of BabyAGI," re-expressing that task-loop as reactive behaviors over a shared graph.

### Contributions (four architectural claims):

1. An event-sourced agent model where graph state is a deterministic fold over an append-only log
2. A determinism contract and replay mechanism with content-addressed cache for model/tool responses
3. A forking and structural-diff primitive for counterfactual exploration
4. A worked, fully reproducible diligence example (Sections 2–6)

The paper explicitly avoids claiming improved task accuracy — "the contribution is the substrate and its guarantees."

## 2. Motivation: Built Around the Model vs. Built on the Log

Conventional architectures scatter state across different locations — "the objective in a prompt, the rules in code or a system message, the tool calls in a framework's internal state, the trace in a transcript, the outputs in a database." ActiveGraph "collapses these into one substrate."

Questions that become trivial under this model: "Why is this fact in the agent's working set? What did the agent believe before it changed rule R? What would have happened had it taken the other branch?"

The paper notes that event sourcing and reactive dataflow are "established patterns in data systems" — the contribution is applying them to "long-running agentic systems" where the payoff is "non-obvious and, we argue, unusually valuable."

## 3. Architecture

### The graph as a projection of the log

The runtime holds an append-only event sequence. Graph state is "computed by folding the event log forward." Two replays of the same log produce "deterministically identical state."

### Events

Every event carries: id, type, payload, actor, optional `caused_by` pointer (linking to the triggering event), and a timestamp. A real run's opening events are shown in Listing 1 — starting with `pack.loaded`, then `goal.created`, `behavior.started`, `object.created`, and `behavior.completed`. Each object includes a "provenance" block naming the behavior and causing event.

### Behaviors

A behavior declares a subscription (event type + optional predicate + graph-shape pattern in a Cypher subset) and a body. The runtime fires the body when a matching change lands. Bodies come in four forms:

- Plain function
- Class (for configurable behaviors)
- LLM-backed routine (request/response logged as events)
- **Relation-behavior** — logic attached to typed edges, so relating two objects can itself carry computation

### Why a graph (not just a log)

Three reasons: subscriptions can be "graph-shape patterns, not just event-type filters"; relation-behaviors allow computation to attach to *relationships*; structural diff is "well-defined precisely because the projection is a graph."

A table compares Conventional agent loops, Memory-layer systems, and ActiveGraph across 8 properties (Persistent state, Provenance, Deterministic state reconstruction, Replay, Fork capability, Fork cost, Structural diff). ActiveGraph is the only system marked "yes" on the last six.

### No workflow — but not no coordination

The diligence example produces "93 objects and 76 relations across three companies without a single line of orchestration code." Control flow is "an emergent consequence of which events match which subscriptions." The claim is not that coordination disappears but that making it implicit and data-driven makes runs "uniformly replayable, forkable, and inspectable."

### The determinism contract

Behavior bodies must not read random, wall-clock time, or fresh UUIDs directly; must not perform I/O outside the framework's tool/model primitives; must not depend on mutable global state. The contract "is not statically enforced" — violations are caught as divergence errors during replay.

The key insight about LLM calls: "A model call is not a deterministic function of its inputs" so it doesn't satisfy the contract at *first execution*. But during *replay*, the recorded response is served from cache. "Determinism is thus a property of re-projecting a log that already exists, never a claim that running the agent is reproducible."

## 4. Replay and Determinism

### The honest problem: models are not deterministic

ActiveGraph "does not pretend otherwise." Instead it records model and tool responses via a content-addressed cache keyed on a hash of the entire request. During replay, "when a behavior re-fires with a request whose hash matches a recorded one, the cached response is served." No new model call is made. "The cost is disk space bounded by the size of the run."

Listing 2 shows a real `llm.requested` event: temperature pinned to 0.0, top_p at 1.0, deterministic: true, with a prompt hash and estimated cost of $0.001.

### Strict vs. permissive replay

- **Permissive mode** (default): events re-emitted from log; cache serves matching hashes; unmatched prompts get fresh calls
- **Strict mode**: runtime compares live event stream against recorded stream event-by-event; any divergence raises an error. A green strict replay "is a proof that the run is reproducible."

## 5. Forking and Structural Diff

This is described as "the property that most distinguishes ActiveGraph from conventional agent frameworks" — the ability to ask cheaply and honestly "what would have happened if I had done this differently?"

### Forks are cheap

The shared prefix is not recomputed; all model/tool responses for those events come from the content-addressed cache. "A fork of a 200-step run that changes one setting at step 150 pays only for steps 150 onward." Most agent frameworks "cannot fork at all, because their state is not reconstructable."

### Forks are honest

A fork's relationship to its parent "is the literal sharing of event ids up to the cutoff." Anyone can verify the shared lineage by reading the two logs.

### Forks vs. frames

Frames are a lighter-weight in-run primitive for parallel sub-contexts that reconverge. Decision rule: "if the branches might diverge permanently or need independent persistence, inspection, or diffing, fork; if they are short-lived and reconverge, use a frame."

## 6. Worked Example: A Diligence Pack

The reference pack performs investment due diligence — from a company name generates research questions, researches against a document store, extracts claims, detects contradictions, identifies risks, and synthesizes a memo. It "ships with recorded fixtures, so the entire run executes offline, with no API key."

Running `activegraph quickstart` (after `pip install activegraph`) executes the demo on three companies (Northwind Robotics, Stellar Logistics, Pinecone Bio) "requiring no API key and completing in under thirty seconds."

### Key quantitative results:

- **671 events** total
- **93 objects** (3 companies, 24 questions, 9 documents, 25 claims, 25 evidence items, 1 contradiction, 3 risks, 3 memos)
- **76 relations**
- **103 model calls** and **48 tool calls**
- Zero orchestration code

### Lineage as the deliverable

A claim object like a revenue figure "carries a provenance block naming the behavior that created it (document_researcher), the event that caused it, and the specific model request event that produced it." It's linked by typed relations to the question it *addresses*, the document it is *derived_from*, and the evidence that *supports* it. "For diligence, research, compliance, or scientific work ... this recoverable chain from goal to output, reconstructable from the log alone, is the actual product."

## 7. An Architectural Affordance for Self-Improving Agents

The paper explicitly states this section is not an evaluated claim — "we do not present a self-improving agent or measure one."

Key argument: On conventional architectures, self-modification is "dangerous precisely because it is invisible and irreversible." On ActiveGraph, "a self-modification is itself an event," yielding two affordances:

1. **Auditability and rollback:** a run can be replayed as it was before a self-modification
2. **Fork-and-diff as evaluation primitive:** propose a change, fork at the proposal point, run forward, structurally diff against the parent — "because the shared prefix is served from cache, each candidate improvement is evaluated without re-paying for the history that preceded it"

## 8. Related Work

Four areas are discussed:

1. **Agent memory systems** (MemGPT/Letta, Zep/Graphiti, Mem0, Hindsight) — these treat memory as a "layer" rather than the primary substrate; ActiveGraph takes the opposite stance that "the log is primary and everything else ... is a projection of it, so provenance is total and replay is exact"

2. **Event sourcing and reactive dataflow** — borrowed from data-systems practice; the contribution is applying these to agentic systems with nondeterministic model calls

3. **Blackboard architectures** (1970s–80s) — structurally close to behaviors reacting to a shared graph. Two limitations were era-specific: brittle hand-built knowledge sources and hand-authored control logic. "Large language models dissolve both constraints." The paper argues this model "may fit LLMs better than the conversational loop they are usually wrapped in"

4. **BabyAGI lineage** — ActiveGraph is a "direct architectural successor" making every step "durable, inspectable, and replayable"

## 9. Limitations and Conclusion

### Failure modes (candidly enumerated):

- **Divergence/looping** — behaviors triggering behaviors; defense is a per-run budget (caps on events, behavior calls, model calls, patches, recursion depth, wall-clock time, and cost)
- **Replay cost** grows with log length; no checkpointing or compaction yet for very long runs
- **Schema evolution** — migration tooling exists but is "a real operational burden"
- **External tool side effects** — only the *record* of the mutation replays deterministically, not the mutation itself
- **Concurrent/distributed writers** and multi-agent contention over a shared graph are unresolved

### Costs acknowledged:

The determinism contract "places a real burden on behavior authors and is enforced only dynamically." Determinism consumes "store space proportional to run size." The paper "report[s] no large-scale empirical evaluation of task performance."

### Conclusion:

"A single inversion — treating the append-only event log as the agent rather than as its exhaust — buys a set of properties that are otherwise hard to obtain together." For long-running systems where one must "be able to explain, reproduce, and revise what the agent did, these properties are not conveniences. They are the point."
