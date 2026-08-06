# Context Graphs

Karan Kalra's vendor-blog essay that accidentally names the next frontier of agent infrastructure: structured, graph-based memory that captures not just *what* decisions were made but *why* — and makes that reasoning available to agents as precedent rather than forcing re-derivation on every run.

---

## Core Argument

The essay opens with a $480k renewal negotiation where an agent can see the *outcome* of a past exception (Globex got 20%) but can't access the *reasoning* (leadership approved it because of a strategic bundling deal). The diagnosis: "we have gotten extremely good at recording **what** happened, but we systematically throw away **why** it happened."

The proposed fix is a **context graph** — "a way of structuring an agent's memory as a graph, where nodes hold pieces of information and edges hold the relationships between them." Unlike vector RAG (which returns similar chunks with no connection info), a context graph stores typed edges: "this invoice –follows–> that policy," "this exception –was granted because of–> that strategic deal." The slogan: **"similarity is not relevance."**

## Key Quotes

> "we have gotten extremely good at recording what happened, but we systematically throw away why it happened."

This is the essay's thesis sentence and it lands hard. Every system of record (Salesforce, SAP, Jira) is an append-only log of state changes with the reasoning stripped out. We built the perfect infrastructure for amnesia.

> "A context graph is a way of structuring an agent's memory as a graph, where nodes hold pieces of information and edges hold the relationships between them."

The definition is deceptively simple. What makes it novel is the design target: the graph is "optimized for the agent to read, not for a human to browse." This inverts the usual knowledge-management assumption that human readability matters.

> "capture is a side effect of doing the work"

The most important sentence in the piece. Wikis, Confluence, ADRs, and post-mortems all fail for the same two reasons: writing is friction, and nobody reads. Agents break both: the context is already in the active window at decision time, and the agent is "a tireless reader that will happily consult ten thousand past decisions." This changes the economics of organizational memory from an opt-in chore to a free byproduct.

> "A correction today becomes a rule tomorrow. A trace today becomes precedent next quarter."

Kalra channels the ACE paper's vision of cumulative context as a self-improving playbook. The agent doesn't need fine-tuning; it just needs access to yesterday's decisions and their rationale. This is a genuinely different theory of agent improvement — learning through accumulation of structured memory rather than model updates.

> "Most vendor decks make it sound shipped. It isn't."

Refreshing honesty in a vendor blog post. The field is early, the hard problems (who writes the trace, how to avoid garbage precedent, the decision swamp of contradictory half-truths) are unsolved.

## Key Themes

- **#concept** — Context graphs as structured agent memory with typed edges, distinct from both vector RAG and traditional knowledge graphs
- **#pattern** — Decision traces as the unit of storage: problem, options weighed, rejected alternatives, constraints, exceptions, who decided, reasoning
- **#pattern** — Capture-on-the-write-path: instrument the decision point, not the post-hoc reconstruction
- **#concept** — "Similarity is not relevance" — the core critique of naive vector RAG that the wiki has surfaced from multiple angles ([[Context Rot]], [[Context Engineering at the Frontier (Linus Lee)]])
- **#tool** — The four-layer stack: systems of record → harness → context graph → agents and humans

## Critical Analysis

**What it gets right:** The "what vs. why" diagnosis is genuinely important and under-discussed. Every production agent builder eventually discovers that the hard problem isn't retrieval — it's that the thing you need to retrieve was never captured. Kalra's framing of this as a *write-path* problem, not a search problem, is the right level to pitch the conversation. The ADR analogy is sharp: ADRs were invented in 2011 to solve exactly this, and most ADR folders die at three entries for the same reasons Confluence pages do. The argument that agents change the economics (capture becomes a side effect of doing work; reading becomes free because the agent is tireless) is the most interesting idea here and deserves more development than Kalra gives it.

**What it skips:** The "who writes the trace" problem is the essay's elephant. Kalra acknowledges it ("human typing rationale feels like wiki maintenance all over again; model-inferred rationale puts shaky reasoning in the immutable record") but waves it away. This is *the* problem. A context graph full of hallucinated rationales is worse than no graph at all — it adds confident-looking structure to bullshit. The garbage-in-garbage-precedent problem gets one bullet point but is actually the central design challenge.

**The vendor problem:** This is a Nanonets blog post, and Nanonets sells document-processing automation. The essay's proposed solution (context graphs) is adjacent to but distinct from their core product. The framing serves the narrative that "systems of record can't do this" and "you need an orchestration layer," which conveniently describes where Nanonets wants to play. The argument is still good — but the reader should know whose interests it serves.

**The decision swamp is real:** Kalra's most honest paragraph acknowledges that "a graph of millions of contradictory, half-true traces is the same failure with extra edges." This is the same problem that kills wikis (stale pages nobody deletes) and knowledge graphs (schema drift, entity dedup hell). The essay doesn't solve it, but naming it is useful.

**Relationship to existing ideas:** The context graph sits at the intersection of several things the wiki tracks: event sourcing (append-only logs of decisions with rationale), knowledge graphs (typed entities and edges), agent memory architectures ([[Sawtooth Memory]], [[MELT]], [[Slate]]), and the continuity-over-memory argument ([[Coding Agents Continuity Not Memory]]). What's genuinely new is the *write-path* emphasis — instrumenting the decision point rather than reconstructing context after the fact. This connects to [[State System]]'s evidence-first commits and [[Agent Identity]]'s argument that memory is retrieval but identity is participation. The ACE paper cited (arXiv 2510.04618) deserves its own wiki page.

**The hard part nobody talks about:** Context graphs require the agent to *know it's making a decision worth recording.* That's a classification problem in itself — is this invoice approval routine or precedent-setting? Is this exception one-off or pattern-forming? The essay doesn't address this, but it's the meta-cognitive gate that determines whether the graph fills with signal or noise.

The retrieval-based alternative to context graphs is exemplified by [[Cerebras Knowledge Base Architecture]], which also emphasizes write-time structuring (LLM distillation before embedding) but opts for hybrid retrieval over typed edges. The two approaches converge on the same insight — structure at ingestion time, not search time — but diverge on whether that structure should be edges between entities or enriched embedding documents.

At the minimalist extreme, [[Zero-Mem]] builds an entity-context graph with co-occurrence edges only — no relation types, no inferred connections, no LLM extraction. It proves that even untyped edges, when combined with Personalized PageRank propagation, surface useful cross-turn evidence. The trade-off: typed edges like Context Graphs proposes capture *why*; co-occurrence edges like Zero-Mem uses capture only *that* entities appeared together. The extra structure of typed edges is valuable precisely when the agent needs to understand relationships, not just adjacency.

---

*Sources: [[summary/what-is-a-context-graph]]*
*Last updated: 2026-07-05*
