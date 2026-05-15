# How AI Agent Memory Works

Mert Cobanov's interactive essay is the best single-page introduction to agent memory architecture I've seen. It's the missing textbook chapter between [[Memory Mechanism]]'s taxonomy and [[Agent Memory and Context]]'s landscape survey — practical, visual, and grounded in production realities rather than research papers.

---

## Key Quotes

> "A language model on its own is stateless."

The starting premise. Everything that follows is about the orchestration layer that carries information forward across turns. The central question: "what should we put in the prompt this time?"

> "Context is not a database. No query plan, no index, no access control, no TTL, no conflict resolution."

A sharp corrective to the naive "just stuff everything in the context window" approach. The context window is RAM, not a database. Treating it as storage confuses two fundamentally different substrates.

> "A frustrating agent forgets everything. A dangerous one remembers wrong."

The best one-liner in the piece. It captures why memory governance matters more than memory capacity. Most teams optimize for remembering *more*. They should optimize for remembering *correctly*.

> "Memory governance is what separates a one-off demo from a production agent."

The lifecycle view of memory — write, age, supersede, redact, forget — is the essay's deepest contribution. Naive append leaks PII and creates contradictions. Naive overwrite loses temporal context. Governed memory preserves history, marks superseded facts, redacts sensitive patterns *before* they enter the store.

> "In multi-agent memory, sharing buys collaboration, and grows the attack surface."

The recommended default: private memory by default, shared memory explicit. Six failure modes enumerated: cross-user leakage, over-sharing, poison propagation, conflicting decisions, stale playbook, attribution loss. Prevention: `tenant_id + user_id` filters on every read, default deny across tenants.

---

## Key Themes

#memory #context-engineering #agents #RAG #vectors #multi-agent #production

### The Four Memory Types

Cobanov maps cognitive science categories to agent architectures cleanly:

| Type | Definition | Retrieval Strategy |
|------|-----------|-------------------|
| **Episodic** | Time-stamped events | Recency / date range |
| **Semantic** | Facts and relations | Vector search (RAG) |
| **Procedural** | Learned skills & tools | Tool invocation |
| **Working** | Current scratchpad | In-prompt context |

All four operate simultaneously within a single turn. The user doesn't see the handoff between them.

This maps well to [[Memory Mechanism]]'s five types (session, project, semantic, episodic, procedural) but adds retrieval strategy as the organizing dimension. Cobanov's taxonomy is about *how you fetch*, not just *what you store*.

### The RAG Loop, Actually

The retrieval-augmented generation pipeline runs every turn: receive → embed → vector search → retrieve top-k → compose prompt → LLM answers → govern new info → update memory. Two production tricks:

- **HyDE (Hypothetical Document Embeddings):** Embed a hypothetical answer instead of the question. A question and its answer are different shapes in embedding space.
- **Reciprocal Rank Fusion (RRF):** Combine dense, sparse/BM25, and graph retrievers by merging rankings. Not a weighted average — a rank-based fusion that assumes nothing about score distributions.

[[Context Rot]] addresses the problem HyDE and RRF don't: even good retrieval degrades over time if you don't track whether retrieved context actually helped. Wilson scoring + dynamic weighting shifts from embedding similarity to outcome-based learning.

### Architecture Tradeoffs

Six approaches compared across scale, structure, supersession, PII gating, sharing, and audit:

- **Simple buffer:** FIFO eviction. Zero governance. Works for single-session tools.
- **Rolling summary:** Summarize older turns. Loses detail, saves tokens.
- **Vector store:** Embed everything. Easy to poison if writes are unguarded.
- **Knowledge graph:** Structured relations. High maintenance, high precision.
- **Hierarchical (MemGPT-style):** OS-like memory paging. Complex, powerful.
- **Self-editing (Letta-style):** The agent edits its own memory. Most governed, most complex.

The vector store warning deserves emphasis: "Easy to poison if writes are unguarded." [[NornicDB]] addresses this with Ebbinghaus-based decay — unused memories fade naturally. [[mira-OSS]] addresses it with first-person narrative framing that the model treats as experience, not external records.

### Production Realities

Cobanov's production section is practical where most essays go vague:

- **Latency target:** 800ms p95 for retrieval
- **Storage tiers:** Hot (KV cache, every turn), Warm (vector index, need-based), Cold (object storage, backfill/audit)
- **Minimum API surface:** `POST /memory/events` (append), `POST /memory/search` (hybrid), `DELETE /memory/{id}` (forget with audit lineage)
- **Multi-tenancy:** Namespace per tenant, single collection + payload filter, or tiered hybrid (small tenants share, enterprise gets dedicated)

The "decide if you even need memory" note is wise. Not every agent turn requires retrieval. Calling retrieval when you don't need it adds latency and noise. This echoes [[How Hightouch Built Their Long-Running Agent Harness]]'s fanout pattern — hundreds of parallel cheap calls can be more reliable than maintaining an embedding pipeline.

---

## Critical Analysis

This is the best introductory resource on agent memory I've encountered. It sits at the sweet spot between [[Memory Mechanism]]'s conceptual taxonomy and the individual implementation pages in this wiki. Where [[Memory Mechanism]] says *what* and [[Agent Memory and Context]] says *who's doing what*, Cobanov says *how* — and does it with interactive demos that make embeddings and retrieval tangible.

But there are gaps and biases worth noting:

**The interactive format is both strength and weakness.** The demos are excellent pedagogy. But they also act as a complexity screen — the essay feels complete because the demos are polished, not because the coverage is comprehensive. The "Production" and "Build your own" sections are flagged as works-in-progress. The capstone sandbox is a promise, not a deliverable.

**No treatment of memory quality metrics.** Cobanov describes the RAG loop but never asks: how do you know if retrieval is working? [[Context Rot]]'s Wilson scoring is the only principled approach in the wiki, and this essay doesn't engage with the question at all.

**The governance model assumes a single-agent world.** Multi-agent memory gets one section and a set of failure modes, but the governance model (mark superseded facts, redact PII) doesn't address the hardest problem: what happens when two agents write contradictory facts about the same entity? [[robot.wtf]]'s git-backed shared wiki approach at least surfaces conflicts via merge resolution, but that's a human-in-the-loop solution.

**The four-type taxonomy is clean but static.** Memories don't stay in their assigned bucket. An episodic memory ("the user debugged this error last Tuesday") becomes a semantic memory ("this error pattern means X") once the timestamp fades in relevance. [[NornicDB]]'s Ebbinghaus decay handles this naturally. Cobanov's taxonomy doesn't address promotion between types.

**Missing: the cost of *not* remembering.** The essay focuses on what to store and retrieve. It doesn't address the inverse: what happens when the agent forgets something it should have remembered? This is where [[napkin]]'s markdown scratchpad pattern is instructive — the cost of a forgotten lesson is higher than the cost of a noisy retrieval.

**Where this fits in the wiki's memory landscape:**

| Resource | What It Does Best |
|----------|------------------|
| [[Memory Mechanism]] | Conceptual taxonomy |
| **How AI Agent Memory Works** | Practical introduction with production details |
| [[Agent Memory and Context]] | Landscape survey and synthesis |
| [[Context Rot]] | Retrieval quality metrics |
| [[Three Tier Memory]] | Concrete scale numbers |
| [[mira-OSS]] | Narrative memory implementation |
| [[NornicDB]] | Temporal decay mechanics |
| [[napkin]] | Minimal viable memory |

If someone asked me "where do I start learning about agent memory?", I'd point them here first, then to [[Memory Mechanism]] for the deeper taxonomy, then to [[Agent Memory and Context]] for the landscape.

---

*Sources: [[raw/how-ai-agent-memory-works]]*
*Last updated: 2026-05-15*
