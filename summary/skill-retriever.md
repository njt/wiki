---
url: https://github.com/ChonSong/skill-retriever
title: "Skill Retriever"
author: ChonSong / Hermes Skill Retriever Contributors
date_fetched: 2026-07-08
date_published: 2025
topics:
  - agent-memory-and-context
---

Skill Retriever is a semantic skill-retrieval plugin for Hermes Agent that
pre-filters a corpus of 1,200+ skills (998 community-contributed from
AgentSkillOS plus 211 Hermes-native) down to the top 5 most relevant per user
query. Rather than using vector embeddings, it navigates a pre-built LLM-driven
capability taxonomy — a tree of ~10,000 categories — selecting relevant branches
via recursive LLM calls at each level. Results are injected as natural-language
hints into the user message before it reaches the model.

The central architectural bet is **LLM-as-classifier instead of embeddings**.
Vector similarity can only surface skills that are semantically close in
embedding space; the tree-based approach applies reasoning at each decision
point, finding connections that a pure similarity search would miss. This comes
at the cost of latency (1–3 extra LLM calls per query) and staleness (the tree
must be rebuilt to include new skills), but scales to 10K+ skills where a flat
system prompt would be unusable.

The search algorithm is a recursive tree descent with three key optimizations:
auto-expansion when a node has few children (avoiding unnecessary LLM calls),
early-stopping when a single selected branch contains few enough skills to
collect wholesale, and parallel search of sibling branches via a thread pool. A
final pruning stage uses a **workflow-stage model** (upstream → production →
downstream) to deduplicate and rank results — reasoning about what a complete
task pipeline needs rather than just scoring individual skills for relevance.

Tree construction happens offline in two phases: an LLM assigns all skills to
five hardcoded root categories (content-creation, data-processing, development,
automation, domain-specific), then recursively splits oversized groups. The
builder uses a queue-based parallel pattern — `FIRST_COMPLETED` with a thread
pool — so new sub-groups enter the queue as soon as any split finishes. Graceful
degradation handles LLM failures: if splitting fails, the node becomes a leaf
with all its skills rather than crashing.

The plugin layer is minimal (185 lines). It runs as a Hermes `pre_llm_call`
hook, lazy-loads a searcher singleton on first use, skips messages under 10
characters, and formats results with safety badges (`🔒hermes` for trusted
installed skills, `🌐community` for unreviewed AgentSkillOS skills, `⚠️` for
safety-flagged ones). Credential configuration cascades from dedicated
environment variables through OpenAI defaults, so Hermes users need zero
additional setup.

The codebase is ~4,400 lines of Python 3.10+, MIT-licensed, with 40 tests
across 4 files. It ships with a CLI offering search, build, list, and info
subcommands.
