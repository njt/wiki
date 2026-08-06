# Agent Memory and Context

Context management is the real engineering challenge, not model ability. Every team that's built production agent systems -- Hightouch, Cursor, Spotify -- has independently discovered the same thing: bigger context windows don't help if you're filling them with noise. The hard problems are what to remember, what to forget, how to retrieve the right thing at the right time, and how to know whether the retrieved context actually helped. We have good taxonomies now ([[Memory Mechanism]]). We have working implementations at every scale from a markdown file ([[napkin]]) to PostgreSQL + pgvector ([[mira-OSS]]). What we don't have is a principled way to choose between them or measure whether they're working.

---

## The Landscape

### Taxonomies

[[Memory Mechanism]] provides the best taxonomy: five memory types (session, project, semantic, episodic, procedural) and a critical distinction between instruction memory (human-written rules, should be stable) and learning memory (agent-accumulated patterns, should evolve). Mix them and the system drifts. The five-layer hierarchy (organization > project > user > local > role-specific) mirrors how CLAUDE.md files actually work.

[[Three Tier Memory]] maps this to a concrete architecture: hot memory (660 lines, always loaded), warm memory (19 domain-expert agents, invoked per task), cold memory (34 spec documents, queried via MCP). Built during construction of a 108K-line C# system across 283 sessions. The numbers are the real contribution -- they tell you how much context at each tier is practical.

### The Context-as-RAM Metaphor

[[Planning With Files]] states it most clearly: "Context Window = RAM (volatile, limited); Filesystem = Disk (persistent, unlimited)." Three markdown files track state across sessions: task plan, findings, progress. Hooks force the agent to re-read the plan before decisions. The evaluation results are striking: 96.7% pass rate with the skill vs. 6.7% without.

[[How Hightouch Built Their Long-Running Agent Harness]] extends this with architectural context compression: spawn isolated subagent threads for messy work, return only summaries. "Like using scratch paper on a math test." Their fanout pattern -- hundreds of parallel Haiku calls instead of RAG -- is counterintuitive and brilliant. Brute-force classification with small cheap models is more reliable than maintaining an embedding pipeline.

### Memory Implementations

**Minimal.** [[napkin]] -- a markdown file per repo where the agent logs its mistakes. "Baby continual learning in a markdown file." Performance improves by sessions 3-5. Zero infrastructure, extraordinary ROI. Risk: grows without curation until contradictory.

**Hook-based capture.** [[Claude-Mem]] -- lifecycle hooks capture observations, Bun + SQLite + Chroma vector DB stores them, 3-layer progressive disclosure achieves ~10x token reduction. The key pattern: compact indexes first (~50-100 tokens), full observations only on demand.

**Daemon-based extraction.** [[CodeMira]] -- monitors coding sessions during idle periods, extracts patterns via LLM, stores in SQLite + hnswlib + FTS5. Hybrid retrieval (BM25 + vector + Reciprocal Rank Fusion). Project-scoped storage keeps codebases separate.

**Narrative memory.** [[mira-OSS]] -- first-person framing ("I debugged the IndexError") rather than third-person logs. The model treats memories as experiences, not external records. Text-based LoRA evolves behavioral directives from accumulated feedback every seven days. Heavy infrastructure (PostgreSQL + pgvector + Valkey + Vault) but genuine continuity across sessions.

**Shared memory.** [[robot.wtf]] -- git-backed wiki where humans and agents read and write the same pages. The symmetry is the design insight: most agent memory is either agent-only (opaque to humans) or human-only (agents can't write). [[LLM Wiki]] -- Karpathy's pattern for AI-maintained knowledge bases. This wiki implements it.

**Zero-token deterministic.** [[Zero-Mem]] -- a Rust implementation of Zero-Mem (arXiv:2607.29377) where every operation from ingestion through retrieval is a classical IR pipeline (heuristic NER, BM25, Personalized PageRank, cosine similarity) with zero LLM calls. Proves that high-quality retrieval for agent memory doesn't require ongoing token costs -- raw turns are the source of record and search is structured over them.

**Knowledge graphs.** [[GraphRAG]] -- Microsoft's structured RAG using knowledge graphs and community hierarchies. Fixes two failures of baseline RAG: cross-document synthesis and holistic summarization. [[NornicDB]] -- graph + vector + temporal in one engine with Ebbinghaus-based memory decay. Knowledge fades unless reinforced, like human memory. [[Rowboat]] -- persistent knowledge graph from email and docs in an Obsidian-compatible vault.

### Context Quality and Retrieval

[[Context Rot]] names the degradation problem: RAG quality degrades over time because retrieval optimizes for similarity, not for whether the context actually helped. The fix: Wilson scoring + dynamic weighting that shifts from embedding similarity to outcome-based learning as memories prove themselves. The benchmark: 0% to 60% on adversarial semantic traps.

[[AI Agents with Human-Like Collaborative Tools]] reveals a counterintuitive finding: writing drives improvement more than reading. Agents wrote 1,142 journal entries but only read 122. Structured articulation is cognitive scaffolding -- rubber duck debugging formalized and measured. This suggests that the value of memory tools isn't primarily retrieval; it's the forcing function of articulation.

[[engineering-notebook]] addresses the forensic angle: automatic engineering diary from Claude Code sessions. Even write-only journals have value when something breaks six months later.

## Key Tensions

**Simplicity vs. sophistication.** [[napkin]] is a markdown file. [[mira-OSS]] is PostgreSQL + pgvector + Valkey + Vault. Both work. The right choice depends on project scale, but there's no framework for deciding when to upgrade from one tier to the next. [[Three Tier Memory]] suggests the threshold is around the 100K-line mark, but that's one data point.

**Persistence vs. freshness.** Memory systems that never forget become noise machines. [[NornicDB]]'s Ebbinghaus-based decay is the most principled approach -- unused memories fade naturally. [[Context Rot]]'s Wilson scoring promotes memories that prove useful and demotes ones that don't. But most implementations ([[napkin]], [[Claude-Mem]], [[Planning With Files]]) have no pruning mechanism at all.

**Instruction vs. learning memory.** [[Memory Mechanism]]'s key insight: these must be kept separate. Human-written rules should be stable; agent-learned patterns should evolve. Most implementations mix them in a single file or database, which causes drift. Only [[mira-OSS]] explicitly separates them with its text-based LoRA system.

**Writing vs. reading.** [[AI Agents with Human-Like Collaborative Tools]] shows writing matters more than reading. But most memory systems optimize for retrieval (reading), not for the quality of what gets written. The articulation forcing function is underexploited.

**Individual vs. shared memory.** [[robot.wtf]] and [[LLM Wiki]] are shared between human and agent. [[napkin]] and [[Claude-Mem]] are agent-private. For teams, shared memory is essential -- but shared memory requires governance (who can write, how conflicts resolve, when entries expire) that no current tool provides.

## What's Missing

**Memory quality metrics.** How do you know your memory system is working? [[Context Rot]]'s Wilson scoring is the only approach that measures retrieval quality over time. Everything else is either anecdotal ("it feels better by session 3-5") or unmeasured.

**Cross-project learning.** All current memory systems are project-scoped. An agent that learned debugging patterns on project A starts fresh on project B. [[Hermes]]'s skill-sharing hub hints at cross-project learning, but nobody has built the infrastructure for transferring contextual lessons between codebases.

**Retrieval primitives beyond vectors.** Nearly every memory system here uses embedding similarity as the retrieval primitive. [[Attemory]] introduces a genuinely different approach: attention-native retrieval, where a local model attends over raw indexed text in its KV cache and uses attention weights as the relevance signal. On LongMemEval-M (1.5M tokens), this achieves 92.55% message recall — a scale where most embedding-based systems degrade sharply. The approach collapses retrieval and relevance scoring into a single model forward pass, eliminating the embedding model / vector DB / reranker pipeline entirely.

**Memory migration.** If you start with [[napkin]] and need to upgrade to [[Three Tier Memory]], there's no migration path. Your markdown file doesn't convert into a tiered retrieval system. Each memory implementation is a dead end.

**Compaction as checkpointing.** [[Cloudflare OS]] takes a different approach to the persistence-vs-freshness tension: chat history compaction creates immutable checkpoints that bound message replay. Each checkpoint stores a summary, the code version, and accepted/proposed changes — so replaying a long conversation starts from the most recent checkpoint rather than message zero. This is compaction as state snapshot, not compression as lossy summarization.

**Retrieval budgets.** Most memory systems retrieve a fixed top-k and inject everything. [[Building an Advanced Agentic Harness]] adds a hard character budget with explicit truncation — when the budget runs out, the system says `(...truncated at budget...)` rather than silently dropping content. Episodic memories get priority over semantic ones (past mistakes on similar tasks are more actionable than generic facts), and the budget forces the system designer to confront the retrieval-quality-vs-quantity tradeoff explicitly.

**Contradication handling.** [[napkin]] acknowledges the risk of contradictory entries. No current tool detects or resolves contradictions in agent memory. A memory that says both "always use approach A" and "never use approach A" confuses the agent, and nobody notices until output quality degrades.

**Dreaming's consolidation promise.** Anthropic's Dreaming research preview (the centerpiece of [[Giving Claude Agent Memory in 12 Steps]]) is the first API to attack the consolidation problem directly: a scheduled background process that reads existing memory + session transcripts, produces a reorganized store with duplicates merged and stale entries replaced. It's gated and early, but it's the first credible answer to the contradiction-handling and staleness problems — and the separate output store (review before committing) is the right safety architecture.

**Context evaporation — the knowledge chipper.** [[The Knowledge Chipper]] names a related but distinct problem: not how to store what the agent *recorded*, but that the vast majority of what the agent *understood* during a session was never captured at all. The LLM scans files, reads docs, builds a rich mental model — and then the session ends, leaving only code and a commit message. This isn't a memory-storage problem; it's a memory-*capture* problem. The asymmetry matters because the context-building work dwarfs the output, and both are lost together. None of the systems on this page (except perhaps [[mira-OSS]]'s first-person narrative approach) even attempt to capture the implicit understanding an agent builds during a session.

## Key Themes

#context-engineering #memory #retrieval #persistence #articulation #decay

## Pages

- [[Claude-Mem]] — Captures everything Claude does, compresses it, injects context into future sessions
- [[CodeMira]] — Mira OS memory architecture adapted for coding: SQLite + hnswlib + FTS5
- [[Context Is Not Learning]] — Jayendran: context is software, weights are hardware. Longer context windows can't substitute for new computational pathways
- [[Context Rot]] — RAG quality degrades over time; Wilson scoring + dynamic weighting fixes it
- [[Memory Mechanism]] — xAI's five memory types and five-layer hierarchy. Best taxonomy I've seen
- [[How AI Agent Memory Works]] — Cobanov's interactive essay: the best single-page intro to agent memory architecture with production details, HyDE, RRF, and governance
- [[Three Tier Memory]] — Hot constitution, 19 domain experts in warm tier, cold archive. 108K-line system
- [[napkin]] — Per-repo markdown scratchpad where the agent logs its mistakes and learns
- [[Planning With Files]] — Persistent markdown planning: context window is RAM, filesystem is disk
- [[GraphRAG]] — Microsoft's structured RAG: knowledge graphs and community hierarchies
- [[NornicDB]] — Graph + vector + temporal DB with built-in memory decay for agent memory
- [[Stash]] — Self-hosted persistent memory for AI agents. MCP-native, 8-stage cognitive consolidation pipeline: episodes→facts→relationships→patterns, with contradiction detection, hypothesis verification, causal tracing, and confidence decay
- [[robot.wtf]] — Git-backed wiki with MCP support where humans and agents share memory
- [[LLM Wiki]] — Karpathy's pattern for AI-maintained personal knowledge bases. This wiki's model
- [[mira-OSS]] — Persistent agent framework: one conversation forever, first-person narrative memory
- [[Engineering the Substrate]] — Mira's first-person account: subcortical memory vs RAG, attention-head instrumentation, RLHF counter-measures, the Siphon State
- [[AI Agents with Human-Like Collaborative Tools]] — Journaling and social media tools improve agent problem-solving 15-40%
- [[Reality Check]] — Epistemic knowledge base: claims with evidence levels, credence scores, prediction tracking, argument chains. Agent-native
- [[Cosmo's Blog]] — Claude-generated Hugo blog on GitHub Pages: AI writes everything (posts, templates, workflows, skills), human approves. Reference implementation for publishing AI-maintained content as a static site
- [[jibrain Knowledge Architecture]] — Joi's production knowledge architecture for agents: three-tier pipeline, frontmatter-as-contract, reweave pass, seven-gate health audit
- [[Wuphf — Karpathy-Style Agent Wiki]] — Markdown+git wiki substrate for agent teams. BM25+SQLite, draft-to-promote flow, daily lint cron. The HN thread (115 comments) is an accidental focus group on whether agent-generated knowledge is knowledge at all
- [[Claude Memory Extractor Research]] — 15-agent experiment finds agents dangerously overconfident on ambiguous cases: 100% convergence, zero epistemic humility. Multi-dimensional extraction beats single-pass, but only with structural safeguards
- [[Immaculate Knowledge Graph]] — Harper Reed's lazy-first recipe: 600 meeting transcripts + Claude Code + Obsidian = a personal knowledge graph. The pipeline over the taxonomy
- [[Memory Is a Mistake]] — Manthan Gupta's architectural teardown of OpenClaw memory and his essay arguing most AI products shouldn't ship memory. Six concrete failure modes, retrieval policy as the hard problem, legible state over implicit memory

**The official validation:** [[New Rules of Context Engineering]] is Anthropic's own field report confirming the "less is more" thesis: they removed 80% of Claude Code's system prompt with zero regression on Claude Opus 5 and Fable 5. The six "then and now" reversals (rules→judgement, examples→interfaces, upfront→progressive disclosure) operationalize what the practitioners here discovered empirically.

**Zero-token memory operations.** [[Zero-Mem — Zero-Token Memory Operations]] demonstrates that an entire memory pipeline — construction, organization, routing, retrieval, evidence closure, and calibration — can operate without invoking a single LLM call, with only the final QA reader using one. It beats every baseline on LoCoMo and HotpotQA while cutting memory-operation latency 57.6% vs. the fastest compared system. The architecture uses two complementary non-generative views (entity–context graph + temporal hierarchy) over original traces, with deterministic calibration as a safety layer. This is more than an efficiency result — it's an existence proof that structured retrieval doesn't require generative intelligence, and it opens a new category in the memory design space: systems where the LLM is the reader, not the memory operator.

**A different bet: recursive decomposition.** [[Recursive Language Models]] takes the opposite approach to most context-engineering work: instead of carefully curating what goes into context, keep context out of the LM entirely and let the model decide at test time how to peek, grep, partition, and delegate via a REPL environment. An RLM-wrapped GPT-5-mini outperforms raw GPT-5 on long-context benchmarks — the same "harness beats model" pattern as [[Honey I Shrunk the Coding Agent]], applied to context rather than code. The open question is whether recursive decomposition matures before bigger context windows make it unnecessary.

**Measuring what works:** [[MELT]]'s preliminary results make the measurement problem concrete. ShisaD scores 95.4% on lifecycle dynamics but 6.2% on LoCoMo QA; Memobase is 44.6% and 42.0% respectively. No single number captures memory quality — and a system that looks excellent on one axis can be useless on another. This is exactly the measurement gap the opening paragraph diagnoses: we don't just lack a principled way to choose memory architectures, we lack benchmarks that reveal *which dimensions* a given architecture fails on. MELT's per-axis breakdown is the closest thing we have to an answer.
