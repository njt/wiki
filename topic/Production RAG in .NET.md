# Production RAG in .NET

Jamie Maguire's seven-part field manual for building and operating RAG pipelines in .NET, drawn from real client engagements and documentation-site projects. The series traces the full arc from first integration with Microsoft Agent Framework through chunking strategy, latency diagnosis, retrieval debugging tooling, operations administration, and 26 hard-won production lessons — all in C#/.NET, with a candid account of how Claude Code accelerated development without replacing domain understanding.

---

## Key Quotes

> "RAG is a pattern, not a framework, and the gap between a RAG tutorial and a RAG pipeline that stays healthy in production is substantial."

The thesis that unifies the entire series. Maguire returns to it in every post: tutorials show you the happy path with three sample documents; production involves content drift, index degradation, zombie vectors, and chunking regressions that degrade silently.

> "Most RAG problems are not solved by adding more infrastructure. They're solved by shortening the feedback loop between ingest and retrieval quality."

From the workbench post (Part 4). This insight drove the architectural decision to build `DocIngestion` with a JSON-file-backed vector store before touching Elasticsearch — the same abstraction-first pattern that shows up in [[Cerebras Knowledge Base Architecture]] (unified Postgres embeddings table) and [[Building Reliable Agentic AI Systems]] (start simple, add complexity when retrieval quality demands it).

> "The model wasn't slow. The chunk size wasn't the problem. The pipeline was sending too much because there was no deduplication by document source."

From the latency diagnosis post (Part 3). The fix — `GroupBy(c => c.DocumentUri).SelectMany(g => g.Take(1))` — took seconds to write and cut token counts from ~19,700 to ~12,266. The real work was knowing to look. Maguire's cardinal rule: **token count is the first thing to log in any slow LLM pipeline**. Before questioning the model, the infrastructure, or the chunk size, log what you're actually sending.

> "A zombie is a soft-deleted document that still has vector chunks sitting in the vector index. It is not serving users because the content record is marked as deleted, but it is still polluting the vector space and potentially influencing retrieval."

From the admin tool post (Part 5). Maguire names a failure mode that most RAG systems accumulate silently: the two-index architecture (content + vectors) creates a specific class of drift where deleted content leaves orphaned embeddings. His admin tool surfaces zombie counts, lets you filter for them, and cleans them up with one click.

> "Claude Code materially accelerated the work, not by replacing my understanding of what I was building, but by removing the friction between understanding something and having it implemented."

From the lessons-from-the-trenches post (Part 6). Maguire is explicit about the collaboration model: understand the domain well enough to know what correct looks like, use Claude Code to get there faster, and review output with the same care you'd apply to your own code. "It is a collaboration, not a delegation."

---

## Key Themes

#pattern #RAG #.NET #production-engineering #observability #chunking #vector-search #tool #operations

### 1. The Tutorial-to-Production Gap Is Where the Real Engineering Lives

Maguire's series is organized around failures, not features. Every post starts with something that broke: slow queries, wrong retrievals, silently accumulating zombies, ingestion jobs dying at 3am with no error. The patterns he surfaces — deterministic chunk IDs, deduplication by document URI, collection existence checks, semantic caching with observable hit rates — are responses to real problems, not theoretical best practices. This is the same gap that [[Building Reliable Agentic AI Systems]] diagnoses in pharma and [[Cerebras Knowledge Base Architecture]] addresses at scale: production RAG is an operations problem, not a model problem.

### 2. Chunking Strategy Is the Highest-Leverage Decision

Maguire returns to chunking across multiple posts and it's where most of his debugging time goes. Key findings:
- **Paragraph-aware splitting** beats fixed character counts — preserve sentence boundaries
- **Overlap tokens are not optional** — sentences near chunk boundaries silently drop out of retrieval without them
- **Code blocks need special handling** — chunk them separately, tag with metadata, and apply a maximum size cap even when treating them as atomic units
- **Mixed content types embed poorly** — prose + code in one chunk pulls the vector in conflicting directions
- **HTML must be converted to Markdown before chunking** — raw tags and attributes degrade embedding quality

The `TextChunker.SplitMarkdownParagraphs()` from [[Semantic Kernel]] is his starting point, but he built custom chunkers when document structures demanded it.

### 3. Infrastructure Abstraction as Debugging Accelerator

The `IVectorStore` interface (three methods: `UpsertAsync`, `QueryAsync`, `ListDocumentsAsync`) with two implementations — JSON on disk for iteration, Elasticsearch for production — is the series' most transferable architectural pattern. By deferring infrastructure decisions, Maguire turned retrieval debugging from an ops problem into a fast feedback loop. The same pipeline code, same parser rules, same chunk sizes carry directly into production via a config change. This is the same pattern behind [[Just Brute Force Your Embeddings]]: measure the simple thing before reaching for the complex one.

### 4. Operations Are Not an Afterthought

The admin tool (Part 5) and the 26 lessons (Part 6) make explicit what most RAG discussions skip: the pipeline needs ongoing maintenance. Maguire's operational checklist includes:
- **Zombie detection** — soft-deleted documents with orphaned vectors
- **Orphaned flag audits** — documents marked vectorised with no actual chunks
- **Daily health emails** — automated summaries of document counts, zombie counts, and non-vectorised counts
- **Index freshness monitoring** — alert on staleness, not just job failure
- **Batch queries for admin UIs** — `_msearch` cut page load time by ~80% vs. per-row queries
- **Status flags are caches, not ground truth** — cross-reference against actual vector index contents

### 5. Local RAG as a Distinct Capability

The earliest post in the series (Part 7, September 2024) implements 100% local RAG using Phi-3 with ONNX Runtime and local embeddings via Semantic Kernel — no cloud calls. While the model choice has moved on, the pattern remains relevant for air-gapped environments. Maguire encountered and worked around a bug in Semantic Kernel's ONNX connector for local embeddings, manually loading the BERT model via `AddBertOnnxTextEmbeddingGeneration()`. This connects to the broader [[Local and Open Source Inference]] landscape.

---

## The Series Architecture

| Part | Focus | Key Insight |
|------|-------|-------------|
| 1 | Agent Framework + TextSearchProvider | `string` not `ReadOnlyMemory<float>` for embedding property; `WithAIContextProviderMessageRemoval()` prevents context bloat |
| 2 | Chunking, embedding, production gotchas | Overlap tokens are mandatory; relevance thresholds need per-corpus tuning; semantic caching before it's too late |
| 3 | Diagnosing slow queries | `GroupBy(DocumentUri)` deduplication; token count logging as the first diagnostic; count your LLM calls per request |
| 4 | RAG workbench (DocIngestion) | `IVectorStore` abstraction with JSON + Elasticsearch; retrieval debugging as fast feedback loop |
| 5 | Admin tool (DocManagement) | Two-index architecture; zombie detection; synchronous re-vectorisation with exponential backoff |
| 6 | 26 lessons from the trenches | Content scoping with XPath; deterministic chunk IDs; separate content/vector lifecycles; config over code; batch workloads not web workloads |
| 7 | 100% local RAG with Phi-3 | VolatileMemoryStore; `SaveInformationAsync` vs `SaveReferenceAsync`; ONNX connector workaround |

---

## Critical Analysis

**What's genuinely valuable:** Maguire has done something rare — published a complete, honest account of building and operating RAG pipelines across multiple client engagements, including the things that broke. The `GroupBy` deduplication fix, the zombie concept, the `IVectorStore` abstraction, the operational lessons about status flags lying and ingestion jobs dying silently — these are the things you only learn by running systems in production, and they're almost never in tutorials. The series is strongest when it's diagnostic ("here's what the logs showed") and weakest when it's prescriptive without data ("start at 0.70 for your relevance threshold").

**What's undersold:** Maguire mentions Claude Code as a development accelerator but doesn't quantify it. How much faster was development? What kinds of tasks did Claude Code handle well vs. poorly? The claim that "the parts that went wrong were the parts where I gave it a vague brief and accepted the output without reviewing it carefully" is honest but thin — a concrete example of a vague-brief failure would strengthen the argument considerably.

**The .NET angle is both a strength and limitation.** For .NET shops, this series is invaluable — concrete NuGet packages, C# code samples, Elasticsearch NEST client patterns. For everyone else, the patterns translate but the implementation details don't. The `IVectorStore` pattern, the chunking strategy, and the operational lessons are language-agnostic; the `TextSearchProvider`, `SemanticKernel.Connectors.InMemory`, and NEST-specific code are not.

**What's missing:** Maguire doesn't address evaluation systematically. The eval section in Part 2 (context recall, faithfulness, answer correctness) is solid but brief — a paragraph and an xUnit test snippet. Compare this to the eval discipline in [[Building Reliable Agentic AI Systems]] (three reflection loops) or [[AI Engineering for Developers]] ("if you skip eval, you ship regressions"). For a series about production RAG, a deeper treatment of how to measure retrieval quality over time — not just at initial build — would strengthen it.

**The series as a compound artifact:** The index-page format is itself interesting. Maguire treats his blog series as a living document that grows as he finds new gotchas and builds better tooling. This is the same compound-interest approach to documentation that this wiki practices — update existing pages rather than creating new ones, weave in new findings rather than starting fresh. The series is strongest read as a whole; individual posts lose the through-line about the tutorial-to-production gap.

---

*Sources: [[raw/rag-in-net-the-complete-series]], [[summary/rag-in-net-the-complete-series]]*
*Last updated: 2026-08-07*
