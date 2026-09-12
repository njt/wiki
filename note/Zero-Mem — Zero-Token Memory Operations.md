# Zero-Mem — Zero-Token Memory Operations

Zero-Mem proposes and demonstrates that an LLM agent's memory pipeline — construction, organization, routing, retrieval, evidence closure, and calibration — can operate without invoking a single LLM call, consuming zero LLM input or output tokens. The only LLM-dependent stage is final question answering. Across long-memory and long-context QA benchmarks, this zero-token regime achieves competitive or superior performance while reducing memory-operation latency by 57.6% against the fastest baseline. The headline finding: structured agent memory does not require generated intermediate representations.

---

## Core Architecture

Zero-Mem keeps original interaction traces as the source of record — no summarization, no generated abstractions, no LLM-mediated memory updates. From these traces, it derives two complementary views using only non-generative computation (NER via spaCy, BM25, BGE-M3 embeddings):

**Entity–context graph.** Named entities are extracted from each context unit and connected to their containing units via co-occurrence edges. Adjacent context units are also connected. The graph records *observed* co-occurrence and trace adjacency — it does not generate semantic triples or inferred relations. At query time, entities from the query are matched to graph entities, activation propagates across co-occurrence sentences, and Personalized PageRank distributes evidence over the graph.

**Temporal hierarchy.** Traces are organized at three granularities: turns (atomic utterances), windows (short-range context), and episodes (coherent event regions bounded by semantic continuity and temporal/session boundaries). Retrieval proceeds coarse-to-fine (episodes → windows → turns), preserving conversational locality and session-level state. When a selected turn depends on nearby information, its local span is added.

**Query-conditioned routing.** A lightweight query profile (subject anchors, keywords, answer type, temporal cues, boundary constraints) determines which view receives priority. Relational queries (multi-hop, cross-entity) weight the graph view higher; local queries (temporal, session-scoped) weight the hierarchy higher. Both views always execute; routing controls fusion weights.

**Evidence closure and calibration.** After dual-view fusion, evidence closure supplements candidates with relational connections (graph view) and neighboring context (hierarchy view). Deterministic calibration then removes candidates that violate provenance or boundary constraints, ranks remaining evidence by compatibility, and applies post-reader answer calibration (type checks, format normalization, list pruning) — all without invoking an LLM.

## Key Quotes

> "Can an agent memory system eliminate LLM calls from every operation outside final question answering, while retaining structured access beyond flat similarity retrieval?"

The paper's framing question. The answer — yes, and with *better* results on most benchmarks — makes this more than an efficiency play. It's an existence proof that structured retrieval doesn't require generative intelligence.

> "Zero-Mem does not replace raw histories with generated abstractions. Each derived unit retains its original text together with source identifier, session time, boundary identifier, and other available metadata."

This is the design decision that makes Zero-Mem different from every major memory system. Mem0, A-Mem, MemoryOS, CompassMem, and even the efficiency-oriented SimpleMem all generate intermediate memory representations. Zero-Mem refuses to. The provenance chain from retrieved evidence back to original interaction traces is unbroken by model generation.

> "Zero-Mem invokes no LLM during memory processing and consequently consumes zero LLM input or output tokens, whereas even LightMem, the most token-efficient baseline, consumes more than 0.87 million tokens."

The efficiency headline. But the real punchline is that this zero-token regime *improves* answer quality: 10.0% F1 and 11.5% BLEU-1 gains over GAM, the strongest baseline, while cutting latency 57.6% vs. LightMem. This is not a tradeoff — it's a strictly dominant result on both quality and cost axes.

> "Both [single-view] variants remain substantially below the full model, showing that the two structures provide complementary evidence: the graph connects information distributed across documents, while the hierarchy preserves local and multi-granular context needed to interpret those connections."

The ablation tells the design story: neither the graph nor the hierarchy alone is sufficient. The graph excels at cross-document relational reasoning (HotpotQA); the hierarchy preserves the temporal and local context that makes those connections interpretable. The contribution is the *coordination* of the two views.

## Key Themes

#concept #memory #retrieval #graph #efficiency #zero-token

- **Zero-token memory operations as a new operating regime.** The paper formalizes a category that didn't previously have a name: memory pipelines where nothing outside final QA calls an LLM. This is distinct from "efficient memory" (LightMem, SimpleMem) which *reduces* LLM calls but doesn't eliminate them. It's also distinct from "no memory" approaches (LONG-LLM, RAG) which avoid memory infrastructure entirely but pay for it in retrieval quality.

- **Provenance preservation as a design constraint.** By never generating intermediate memory representations, Zero-Mem avoids the omission, merging, and hallucination risks that attend LLM-mediated memory. Retrieved evidence is always traceable to an observed interaction, not a model-generated memory statement. This connects to [[Context Graphs]]'s concern about "model-inferred rationale puts shaky reasoning in the immutable record" — Zero-Mem sidesteps the problem by never inferring.

- **Two views, one substrate.** The entity–context graph and temporal hierarchy are complementary retrieval structures over the *same* underlying traces. This is a cleaner architecture than systems that maintain separate stores for different memory types (episodic, semantic, procedural). The views resolve to the same provenance-bearing units; they differ only in access pattern.

- **Deterministic calibration as a safety layer.** The pre-reader and post-reader calibration steps are deterministic filters that enforce provenance, temporal, and type constraints without model calls. This is [[Guardrails and Feedback Loops]] applied to memory: deterministic enforcement catches what generated guardrails miss. A scalar answer gets replaced only by a unique type-compatible candidate from the evidence set — no LLM judgment required.

- **Query structure as routing signal.** The routing decision (graph priority vs. hierarchy priority) is based on deterministic query-structure signals — question form, temporal/aggregation requirements, availability of subject anchors — not on an LLM's assessment of "what kind of question is this?" This is [[Reducing Token Spend with Deterministic Workflows]] applied to memory retrieval: the classification that could be an LLM call is instead a lightweight deterministic profile.

## Critical Analysis

**The existence proof matters more than the specific architecture.** Zero-Mem's most important contribution isn't its particular combination of entity–context graph + temporal hierarchy — it's the demonstration that a zero-token memory pipeline can work at all. Before this paper, the operating assumption across the field was that structured memory access *required* some form of LLM-mediated generation, whether for summarization (Mem0, A-Mem), graph construction (CompassMem, Zep), or retrieval planning (SimpleMem). Zero-Mem shows you can get competitive or better results without any of it. This is a category error fix at the field level, not just a new architecture.

**The comparison to GAM is the most interesting result and the least discussed.** GAM (General Agentic Memory via Deep Research) is the strongest baseline on LoCoMo and represents the "just-in-time memory" paradigm: do deep, expensive retrieval at query time rather than maintaining an indexed memory store. Zero-Mem beats GAM by 5.4 F1 points *while using zero LLM tokens for memory*. This is remarkable because GAM's entire thesis is that deep query-time retrieval (which burns LLM tokens) is worth the cost. Zero-Mem suggests the opposite: structured, non-generative retrieval can find better evidence than an LLM doing deep research, and do it faster and cheaper.

**The encoder elephant.** Zero-Mem still uses encoder computation (spaCy NER, BGE-M3 embeddings), which the paper accounts for separately from LLM token consumption. The latency numbers (334.77s total, 0.22s/query) include this encoder cost and still beat every baseline by 57.6%. But as models improve and LLM inference gets cheaper, the relative advantage of avoiding LLM calls shrinks. The durable contribution is the architecture, not the specific cost numbers.

**What's missing: contradiction across traces.** Zero-Mem's calibration step discards evidence that violates provenance or boundary constraints, but doesn't address the harder problem of traces that *contradict each other* — one interaction says X, another says not-X, both are valid provenance. [[Agent Memory and Context]] flags contradiction handling as a major gap in current memory systems. Zero-Mem doesn't solve it, and its deterministic calibration would need extension to handle it.

**What's missing: write-path instrumentation.** Zero-Mem assumes traces are already captured. It doesn't address *which* interactions to record, how to structure them at capture time, or how to handle the classification problem [[Context Graphs]] identifies: is this interaction routine or precedent-setting? This is a retrieval-only paper; the full memory lifecycle includes write-path decisions it doesn't engage.

**The spaCy NER bottleneck.** Entity extraction uses an off-the-shelf NER model. For domains with specialized entity types (code, medicine, law), this is a real limitation. The architecture could accommodate domain-specific entity extractors, but the paper doesn't explore this. The entity–context graph is only as good as the entity recognition that feeds it.

**Relationship to the wiki's memory conversation.** Zero-Mem lands in the middle of several ongoing threads:
- It complicates [[Memory Is a Mistake]]: if memory operations are zero-token, the cost argument against shipping memory weakens considerably. The remaining argument is retrieval policy quality, not operational cost.
- It extends [[Context Graphs]]: Kalra's typed-edge context graph is generated by an LLM; Zero-Mem's entity–context graph is observed from traces. Both are graphs over memory, but the *source* of edges is fundamentally different — generated vs. observed.
- It operationalizes [[Reducing Token Spend with Deterministic Workflows]] for the memory domain: identify which parts of the pipeline don't need intelligence, extract them into deterministic code. Jacob's "LLM-as-compiler, not LLM-as-runtime" maps directly to Zero-Mem's "LLM as final reader, not memory operator."
- It provides a new data point for [[Agent Memory and Context]]'s opening diagnosis: the hard problems are what to remember, what to forget, and how to retrieve. Zero-Mem solves retrieval (dual-view, calibrated) without solving the others — it's a retrieval architecture, not a full memory lifecycle.

---

*Sources: [[raw/2607-29377v1]], [[summary/2607-29377v1]]*
*Last updated: 2026-08-06*
