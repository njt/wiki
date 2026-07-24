# Cerebras Knowledge Base Architecture

Cerebras's detailed engineering postmortem of Cerebras Knowledge, an internal RAG system handling 15,000+ queries per day from employees, automations, and AI agents — built in three months and running across chip design, data centers, training, inference, and cloud platform domains. The article is unusually honest about what worked and what didn't, and the X thread that announced it attracted several responses that are independently worth reading.

---

## Architecture

Three layers, all converging on a single Postgres embeddings table with a common schema:

1. **Collection** — ingests from Slack, Google Docs, GitHub, Jira, and custom databases in-place (no migration)
2. **Querying** — planner → executor → RRF fusion → reranker → context expansion → synthesis
3. **Auth & audit** — authentication, authorization, per-query analytics

### Design Philosophy: Meet Data Where It Lives

> Don't migrate data — extract in place from existing platforms.

Cerebras explicitly rejected consolidating everything into one platform. Information stays in native tools; the KB extracts from each directly. This is the opposite of the "single source of truth" instinct most enterprise KB projects start with.

### The Slack Problem

Slack was the hardest data source. Raw transcripts embedded directly performed poorly — too much filler, too many rare tokens lost in embedding space. The solution is a four-signal hybrid retrieval stack that's the most interesting part of the system:

| Signal | What it catches |
|---|---|
| **Full-text search** | Exact error strings, flag names, hostnames that embeddings blur |
| **Embedding search** | Paraphrases like "restore hangs" ↔ "checkpoint stalls on NFS" |
| **Inverse Document Frequency (IDF)** | Rare config flags get boosted; "sounds good, thanks!" gets suppressed |
| **Age decay** | Newer threads win ties; 6-month-old answers may describe dead infra |

Before embedding, raw Slack threads go through **LLM distillation**: an LLM extracts structured fields (searchable question, summary, resolution, systems, code references) and the normalized document is embedded at 3,072 dimensions. The article reports "significant accuracy gains vs embedding raw transcripts."

Long threads get **burst**: consecutive messages from one author are split into per-author bursts, individually embedded if they pass quality gates (IDF ≥ 4.0, combined length ≥ 200 chars). This prevents answers buried deep in long threads from being invisible.

### Code Repositories

Uses CocoIndex for language-aware recursive chunking (class → method → smaller blocks), supporting repos up to 40 GB+. Incremental sync re-embeds only changed chunks per commit.

### Query Pipeline

1. **Planner** — lightweight LLM selects relevant retrieval tools based on project scope
2. **Executor** — parallel fan-out; results normalized to shared evidence schema
3. **Reciprocal Rank Fusion** — `score(d) = Σ 1/(60 + rank)` with smoothing constant k=60
4. **Reranker** — small model scores 0–10, keeps top 10
5. **Context expansion** — neighboring sections pulled in for complete snippets
6. **Synthesis** — final LLM produces cited answer with caveats

### Two Interfaces, Two Audiences

- **Web UI**: Full pipeline, one cited answer per question — for humans
- **MCP**: LLM-free retrieval primitives (`search_slack`, `search_code`, `who_knows`) — for agents like Claude Code to orchestrate themselves

The MCP interface is the clever bit: it lets external agents compose retrieval steps rather than trusting the system's built-in planner, while the KB owns the retrieval quality.

### Projects: Scoped Search

As the corpus grew, global search degraded. Projects bundle Slack channels, repos, and doc spaces per team (Compiler, ML training infra, Data Center Ops). New hires pick a default project at onboarding so their first queries return high-signal results without knowing the company's channel topology.

### Custom Data Sources

Teams submit a PR with a small Python plugin that emits rows matching the embeddings schema. No special handling — "the data queries alongside Slack, code, and docs immediately."

---

## Key Quotes

> The system handles 15,000+ daily queries from three consumer types: human employees, automated workflows, and AI agents.

The three-consumer framing matters. Most KB projects design for one audience (usually humans); Cerebras designed for three from the start. The MCP interface is what makes the agent use case real, not aspirational.

> All data flows into the same embeddings table with a common schema.

This is the architectural bet: a single table, not per-source indexes. It's simple and it works — until it doesn't. The Projects feature is the admission that "search everything" degrades as the corpus grows.

> Raw Slack transcripts are not embedded directly. An LLM extracts structured fields first.

The dirtiest secret in enterprise RAG. Raw text → embeddings is a toy pipeline. Real production systems pre-structure at write-time. This is the same insight as [[Context Engineering at the Frontier (Linus Lee)]]: context engineering IS search engineering, and the write path matters more than the read path.

---

## Critical Analysis

**What's genuinely novel**: The four-signal hybrid retrieval for Slack is the most thoughtful treatment of conversational data in a RAG system I've seen. Most teams give up on Slack or treat it as a second-class source. Cerebras made it the marquee feature and published enough detail to be replicable.

**What's underbaked**: The auth and audit layer gets one sentence. As Terry Li noted, when a knowledge base serves agents that can act on retrieved information, the retrieval quality problem becomes a *control* problem. Kirk Patrick's reply thread goes much further — formal provenance, empty-never-nearest contracts, typed abstention. Cerebras mentions these as future work; the gap between what they shipped and what Kirk describes is where the real engineering lives.

**The ontology elephant**: Kevin Simback's reply is the most important critique in the thread. Retrieval-based systems find text that looks relevant; they don't resolve claims like "why was this invoice paid when it didn't match the PO?" — that requires a derived fact layer over raw communications. Cerebras's architecture handles "what do we know about X?" well; it doesn't handle "is X still true, according to whom, and where do accounts disagree?" This is the same critique [[Context Graphs]] makes: the write path should capture *why*, not just *what*.

**The convergent-evolution signal**: Kirk Patrick and the Hyperspell founder both report landing on nearly identical architectures independently. That's real validation. When three teams converge on unified Postgres embeddings + hybrid retrieval + reranker + MCP, that's not coincidence — it's the local maximum of current technology.

**What the thread reveals about the industry**: The replies are a map of where enterprise knowledge infrastructure is heading. ACL-aware retrieval, ontology-vs-retrieval, agent-native interfaces, evaluation methods, cost modeling — these are the live problems, and nobody has solved all of them. Cerebras solved enough to ship something useful at scale.

---

## Key Themes

- #pattern — hybrid retrieval (full-text + embeddings + IDF + age decay) as the production Slack strategy
- #pattern — LLM distillation at ingestion time rather than raw-text embedding
- #pattern — MCP as the agent-native interface, with LLM-free retrieval primitives
- #pattern — scoped search via Projects to prevent corpus degradation
- #concept — "meet data where it lives" as anti-ETL philosophy
- #concept — convergent evolution: Postgres embeddings + hybrid retrieval + reranker + MCP
- #tool — CocoIndex for code repository chunking
- #tool — Reciprocal Rank Fusion with k=60

---

*Sources: [[raw/cerebras-knowledge-base]], https://www.cerebras.ai/blog/how-we-built-our-knowledge-base*
*Last updated: 2026-07-25*
