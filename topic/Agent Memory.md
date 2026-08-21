# Agent Memory

Angie Jones's definitive seven-type taxonomy of agent memory, informed by her conversations with Oracle's Richmond Alake. The cleanest production-oriented breakdown of memory types available, paired with a concrete reference implementation (Oracle's OAMP) and the sharpest articulation of why judgment — not storage — is the hard problem.

---

## The Taxonomy

Jones distinguishes seven memory types that most practitioners lump into one:

**Conversational memory** — message history. The naive approach everyone starts with (append to prompt) and the one Jones explicitly rejects as "not really memory engineering."

**Semantic memory** — durable facts that outlive specific conversations. User preferences, system requirements. Retrieved by meaning via vector search.

**Episodic memory** — "the 'what happened' layer." Events, workflows, sequences. Benefits from structured storage because ordering matters and vector similarity alone misses temporal relationships.

**Procedural memory** — reusable *how-to* knowledge. Approaches and processes that apply across situations. The most under-exploited type in current systems.

**Entity memory** — facts scoped to specific people, accounts, projects, or objects. Requires filtered retrieval: "What do we know about Acme Corp?" should not return every memory in the system.

**Working memory** — short-term scratchpad for the current task. The filtering problem: not everything in working memory deserves permanent storage, or "the memory store gets noisy very quickly."

**Summary memory** — compressed representations of long threads. The type most users are familiar with because it's what chat interfaces already do.

## Key Quotes

> "LLMs are stateless by design, meaning they have no memory or awareness of past interactions."

The premise that makes all of this necessary. Memory is infrastructure, not a model capability.

> "The most common first attempt is to keep appending prior messages to the prompt. This is not really memory engineering."

Jones draws a sharp line between context-stuffing and architecture. Most agent memory products are still on the wrong side.

> "Real agent memory requires deciding what should be stored, where it should be stored, how it should be retrieved, and when it should be updated or forgotten."

The full lifecycle: write, retrieve, update, forget. Most implementations only handle the first two.

> "The hard part is judgment, not storage."

The single most important sentence in the article. Anyone can build a vector database. Almost nobody knows what belongs in it. This aligns with [[Memory Is a Mistake]]'s argument that most products shouldn't ship memory at all.

> "Bad memory is worse than no memory."

A confident wrong answer is more dangerous than admitting ignorance. This is the same insight as [[How AI Agent Memory Works]]'s "a frustrating agent forgets everything. A dangerous one remembers wrong."

> "You can teach it by turning your system's metadata into memory."

The metadata-as-memory pattern: scan your own database catalog, convert technical schema into natural-language facts, store as retrievable memory. Inverts the usual RAG pattern — the agent generates its own documentation from system metadata.

## Oracle's OAMP

The Oracle AI Agent Memory Package is built on Oracle AI Database 26ai, integrating embeddings, JSON, text search, and SQL in one database. Four design choices worth noting:

1. **Context cards** — instead of dumping raw thread history into the prompt, `get_context_card()` produces a compact, structured block of relevant memory. Prompt = system instruction + context card + user query. The cleanest production answer to "how do you actually inject memory?"

2. **User/agent ownership** — every memory is scoped to a user and agent ID, preventing cross-contamination. This is the `tenant_id + user_id` filtering that [[How AI Agent Memory Works]] recommends as default-deny.

3. **Automatic extraction** — configurable extraction from conversation messages (`memory_extraction_frequency`, `enable_context_summary`). This handles the *write* side of memory, which most open-source implementations ignore entirely.

4. **Database-backed persistence** — vector search alone isn't enough. OAMP uses SQL for filtering, ordering, and structured queries alongside embeddings. This echoes the hybrid retrieval approach in [[Context Rot]] and the database-native pattern in [[Databases and Data]].

5. **Attention-native retrieval as a distinct memory retrieval paradigm.** Jones's taxonomy covers storage and retrieval patterns built on embeddings and structured queries. [[Attemory]] introduces a qualitatively different retrieval primitive: model attention over raw KV-cached text, with no embeddings at all. On LongMemEval-M (1.5M tokens), this achieves 92.55% message recall — competitive with the best embedding-based systems while using a fundamentally different mechanism. It suggests that retrieval quality isn't just about better embeddings; it's about giving the model direct attention over the source text rather than compressed surrogates.

## Critical Analysis

**The taxonomy is the real contribution.** Jones's seven types are more granular and production-grounded than [[Memory Mechanism]]'s five (session, project, semantic, episodic, procedural). Entity memory and working memory fill real gaps that xAI's taxonomy glosses over. Summary memory recognizes that compression is a distinct operation, not just "less context." The seven-type framework should become the standard reference.

**The Oracle section is both strength and weakness.** OAMP is genuinely interesting — context cards and automatic extraction are design choices every memory system should consider. But the article reads like a vendor case study in its second half. The code examples are Oracle-specific; the database dependency is non-trivial. A Postgres + pgvector implementation of the same ideas would serve more readers.

**The "judgment over storage" thesis is correct but incomplete.** Jones is right that deciding what to remember is harder than storing it. But she doesn't address the *evaluation* problem: how do you know whether your memory system is working? [[MELT]] provides a benchmark harness for exactly this, and [[Context Rot]]'s Wilson scoring measures retrieval quality over time. A memory taxonomy without a measurement framework is half the problem.

**Working memory is the most neglected type.** Jones identifies it but doesn't develop it. Working memory — the short-term scratchpad — is where most agent errors actually happen. An agent that misremembers what it's currently doing is worse than one that forgets a user preference from last week. [[Slate]]'s thread-and-episode architecture and [[Sawtooth Memory]]'s L1 working tier address this, but it's still the least mature area of memory engineering.

**The metadata-as-memory pattern deserves more attention.** Teaching an agent about a private database by scanning catalog tables and converting to natural-language memories is a genuinely novel idea that Jones only sketches. It's a form of automated documentation generation that serves as memory bootstrap — and it generalizes beyond databases to any system with structured metadata (APIs, file systems, CI/CD pipelines). Someone should build this as an open-source tool.

**The Postgres + pgvector implementation exists, in a code review agent.** The gap flagged above — that OAMP's code examples are Oracle-specific and a Postgres implementation "would serve more readers" — is filled by [[Building a Production AI PR Review Agent]], which derives three of Jones's types (semantic = code embeddings, episodic = past reviews and disputes, procedural = team conventions) from what a human reviewer does, then collapses all three plus a time-series event spine onto one managed Postgres: pgvector for embeddings, pgvectorscale/DiskANN for index-on-SSD at scale, TimescaleDB hypertables for the append-only span log, continuous aggregates for the cost dashboard. The consolidation argument is worth taking seriously against the multi-store default — three databases means three sets of backups, connection pools, and corruption modes, and answering one question ("for this PR, what did we retrieve, what did we produce, what did it cost?") means stitching three systems together in application code. The caveat: the store chosen is the course's sponsor.

**Where this fits.** Jones provides the cleanest production taxonomy (seven memory *types*). [[Memory Mechanism]] provides the best theoretical framework (especially the instruction vs. learning memory distinction). [[How AI Agent Memory Works]] provides the best interactive introduction with production details. [[Agent Memory and Context]] provides the landscape survey. [[Giving Claude Agent Memory in 12 Steps]] provides the complementary implementation ladder (four memory *layers*: Chat Memory → Projects → CLAUDE.md → Dreaming). Taxonomy and implementation ladder answer different questions — "what kind of memory?" vs. "how do I build it?" — and together they form a complete reference.

## Key Themes

#memory #agents #context-engineering #taxonomy #oracle #database #vector-search #retrieval #rag

---

*Sources: [[summary/agent-memory]]*
*Last updated: 2026-07-05*
