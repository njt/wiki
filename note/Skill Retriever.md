# Skill Retriever

A semantic skill retrieval plugin for the Hermes Agent that replaces the flat skill catalog with a search engine. Uses an LLM-navigated 10,000-category capability taxonomy to surface the top-5 most relevant skills per query — including skills that embedding similarity would miss. Runs as a zero-configuration pre_llm_call hook that borrows Hermes' existing LLM credentials.

---

## Architecture

The system comprises three major subsystems in ~4,400 lines of Python:

**Plugin layer** (`plugin/__init__.py`) — a single Hermes `pre_llm_call` hook. Checks a disable flag, skips short messages, lazy-loads the Searcher singleton, formats top-5 results as a `[Skill Retrieval Hint]` block prepended to the user message. The LLM may then call `skill_view(name)` to load any suggested skill.

**Search engine** (`search/searcher.py`, 733 lines) — recursive LLM-guided descent through a pre-built YAML capability tree. At each level, the LLM selects relevant child nodes from a constrained set of options. Selected branches are searched in parallel via `ThreadPoolExecutor`. A final pruning step ranks and deduplicates skills using a workflow-stage model (upstream preparation → production creation → downstream delivery).

**Tree builder** (`tree/builder.py`, 654 lines) — offline LLM-driven taxonomy construction from a corpus of SKILL.md files. Uses a queue-based `ThreadPoolExecutor` with `FIRST_COMPLETED` pattern: as soon as any node split completes, its children immediately join the queue. Skills are first assigned to 5 fixed root categories (Content Creation, Data Processing, Development, Automation, Domain Specific), then recursively split by an LLM until all groups fit within thresholds.

The tree is stored as YAML and loaded once at search time. Skill metadata (github_url, stars, author) is enriched from a companion `skills.json` at load time, keeping the tree file itself lightweight.

## Key Techniques

**LLM-as-classifier at each tree level, not embeddings** — this is the central architectural bet. Instead of computing vector similarity between query and skill descriptions, the system asks the LLM to *decide* which branches are relevant at each tree node. This means the LLM applies semantic reasoning rather than statistical similarity at each decision point. A skill that helps with task X might not look similar to a query about X in embedding space, but the LLM can reason that it's functionally needed.

**Single-parameter configuration scaling** — `DynamicTreeConfig` derives all thresholds from one `branching_factor` value using multiplicative factors. `max_skills_per_node = 1.5×`, `expand_threshold = 0.7×`, `early_stop_skill_count = 1.7×`. Change one number and all thresholds scale proportionally — no stale magic numbers as the tree grows.

**Queue-based parallel tree construction with FIRST_COMPLETED** — unlike a barrier-based approach where all siblings must finish before the next level starts, the builder uses `wait(futures, return_when=FIRST_COMPLETED)`. Completed splits feed their children back into the queue immediately, maximizing parallelism regardless of split latency variance.

**Workflow-stage pruning model** — the pruning prompt (`prompts.py:188-246`) decomposes tasks into upstream/production/downstream stages. It explicitly warns against keyword-matching: `"promote" ≠ only marketing/SEO skills` — effective promotion requires assets to promote, so creation tools often matter more than distribution tools. This is a deliberate anti-RAG design: relevance is judged by workflow fit, not textual similarity.

**Borrow-mode credential cascade** — the plugin reads `SKILL_RETRIEVER_LLM_*` first, then falls back to `OPENAI_*` env vars. Users already running Hermes with OpenAI need zero additional configuration. The trade-off: the retrieval gate is coupled to Hermes' model provider.

**Event-driven observability** — the Searcher emits events at every stage (`search_start`, `node_enter`, `children_selected`, `skills_selected`, `early_stop`, `prune_start`, `prune_complete`, `search_complete`) through a pluggable callback. Enables WebUI visualization and debugging without coupling to any specific frontend.

**Dual scanner architecture** — separate implementations for the offline corpus scanner (`tree/skill_scanner.py`, reads SKILL.md files from a data directory with metadata enrichment) and the live Hermes scanner (`scanner.py`, reads `~/.hermes/skills/` with category-from-directory inference). The tree uses the former, the plugin uses the latter.

## Design Decisions

**Why not embeddings?** — The README positions this explicitly against pure semantic retrieval. Embedding similarity is narrow and myopic; skills that are functionally relevant but textually dissimilar get missed. The tree provides structure that guides the LLM's reasoning. This is the same insight behind [[Context Graphs]]: "similarity is not relevance."

**Build-once, search-many** — The tree is constructed offline (expensive, many LLM calls) and loaded from YAML at search time. Search uses cheap per-level LLM calls (select N from a constrained set, output a JSON array). This works for a curated skill corpus but wouldn't work for frequently-changing content.

**Fixed root categories, flexible sub-trees** — The top level is hardcoded to 5 categories, providing a stable entry point. Below that, the LLM discovers sub-categories dynamically. This is a deliberate hybrid: human stability at the entry point, LLM flexibility in the details.

**Latency-for-token trade-off** — Each query costs 1-3 additional LLM calls for tree traversal, versus Hermes' OOTB approach of listing all skills in the system prompt every turn. For small skill sets, the OOTB approach wins on latency. For large skill sets, skill-retriever wins on context cost — the break-even is around 200 skills.

**Static tree, no incremental updates** — Skills added after the tree is built won't appear until a rebuild. The data directory is gitignored (240MB) and downloaded separately. This is pragmatic for a plugin but means deployment requires coordinating tree rebuilds with skill corpus updates.

## Comparison Notes

**vs. Hermes OOTB skill discovery**: Hermes lists every installed skill in the `<available_skills>` block of the system prompt every turn. This is zero-latency but burns tokens even on irrelevant skills. skill-retriever transforms discovery from "read the catalog" to "search for what you need" — the trade-off is +1-3 cheap LLM calls per turn vs. constant system prompt bloat.

**vs. [[Elysia]]**: Both constrain tool choice per node rather than dumping all tools into context. Elysia uses a decision tree with hardcoded rules; skill-retriever uses an LLM-navigated taxonomy. The difference: Elysia's approach is deterministic and zero-cost; skill-retriever's is more flexible but costs LLM calls.

**vs. RAG/embedding retrieval**: skill-retriever uses LLM reasoning instead of vector similarity. This helps when a skill's textual description doesn't capture its functional relevance, but it's slower and more expensive than an embedding lookup. The right choice depends on whether your skill set is large enough that precision matters more than speed.

**vs. [[Dippin (Language)]]**: Dippin defines agent workflows as an indentation-sensitive DSL with a compiler pipeline. skill-retriever's workflow-stage pruning prompt achieves a similar effect (thinking in terms of upstream→production→downstream) but entirely within a single LLM prompt rather than a separate language and compiler.

**vs. [[The Agentic Product Standard v2.0]]**: skill-retriever is a concrete implementation of one axis of the standard — dynamic tool selection. It demonstrates the "retrieval over catalog" pattern for skill/tool discovery at scale.

---

*Sources: [[raw/skill-retriever]]*
*Last updated: 2026-07-08*
