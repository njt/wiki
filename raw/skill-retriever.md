---
url: https://github.com/ChonSong/skill-retriever
title: Skill Retriever
author: ChonSong / Hermes Skill Retriever Contributors
date_fetched: 2026-07-08
date_published: 2025
---

# Skill Retriever

AgentSkillOS-powered semantic skill retrieval plugin for Hermes Agent. Pre-filters 1,200+ skills (998 community + 211 Hermes) organized in a 10,000-category capability taxonomy to the top-5 most relevant per query, injected as natural-language hints into the user message.

**Language:** Python 3.10+, 4,400 lines of source
**License:** MIT
**Version:** 0.2.0
**Dependencies:** chromadb, litellm, pyyaml, python-dotenv, rich

---

## Architecture

### High-Level Flow

```
User Query → pre_llm_call hook (plugin/__init__.py)
  → Searcher.search() (search/searcher.py)
    → Load YAML capability tree
    → Recursive LLM-guided node selection
    → Parallel child branch search (ThreadPoolExecutor)
    → LLM pruning (dedup + rank by workflow stages)
  → Hint injection into user message
```

Three major subsystems:

1. **Plugin layer** (`plugin/__init__.py`, 185 lines): Hermes pre_llm_call hook. Minimal — checks disable flag, skips short messages, lazy-loads Searcher singleton, formats top-5 results as `[Skill Retrieval Hint]` block.

2. **Search engine** (`search/searcher.py`, 733 lines): The core. Recursive tree descent with LLM node selection, ThreadPoolExecutor parallelism, LLM pruning with workflow-stage ranking.

3. **Tree builder** (`tree/builder.py`, 654 lines): Offline LLM-driven taxonomy construction from SKILL.md files. Queue-based parallel processing with FIRST_COMPLETED pattern. Outputs YAML + HTML visualization.

### Directory Layout

```
skill-retriever/
├── plugin/__init__.py          ← Hermes pre_llm_call hook
├── plugin/plugin.yaml          ← Plugin manifest
├── src/skill_retriever/
│   ├── __init__.py             ← Public API
│   ├── __main__.py             ← CLI entry point
│   ├── cli.py                  ← search, build, list, info commands
│   ├── config.py               ← Env-based configuration
│   ├── scanner.py              ← Hermes skills scanner (for plugin)
│   ├── search/searcher.py      ← Core search engine (733 lines)
│   ├── tree/
│   │   ├── builder.py          ← Tree builder (654 lines)
│   │   ├── prompts.py          ← All LLM prompts (centralized)
│   │   ├── schema.py           ← TreeNode, Skill, DynamicTreeConfig
│   │   ├── skill_scanner.py    ← Corpus SKILL.md scanner
│   │   └── visualizer.py       ← HTML tree visualization (1094 lines)
│   └── capability_tree/        ← Pre-built YAML + HTML trees
├── data/skill_seeds/skills.json ← Skill metadata (gitignored, 240MB)
├── tests/                      ← 40 tests across 4 files
└── scripts/install.sh          ← One-click Hermes plugin install
```

---

## Core Data Model

### TreeNode (schema.py:98-166)

Generic tree node supporting arbitrary depth:
- **Intermediate nodes**: Have children (no skills directly)
- **Leaf nodes**: Have skills (no children)
- `is_leaf` property: checks `len(self.children) == 0`
- Recursive operations: `count_all_skills()`, `collect_all_skills()`, `get_leaf_nodes()`
- Supports two deserialization formats:
  - `from_recursive_tree()`: New format (`{id, name, description, children, skills}`)
  - `from_capability_tree()`: Legacy format (`{domains: {domain: {types: {skills}}}}`)

### Skill (schema.py:82-95)

Flat dataclass with id, name, description, path, skill_path, content, plus metadata fields (github_url, stars, is_official, author) populated from skills.json at tree load time.

### DynamicTreeConfig (schema.py:35-77)

Single-parameter configuration: `branching_factor` (default 8) derives all others via multiplication:
- `max_skills_per_node` = branching_factor × 1.5 (12)
- `expand_threshold` = branching_factor × 0.7 (5)
- `early_stop_skill_count` = branching_factor × 1.7 (13)
- `lazy_split_threshold` = max_skills_per_node × 1.3
- `classification_batch_size` = branching_factor × 6
- `structure_sample_size` = branching_factor × 12

This proportional scaling ensures derived values stay coherent as tree size changes.

---

## Search Algorithm (searcher.py)

### Recursive Descent (`_search_node`, line 251)

At each node:
1. **Leaf node** (has skills, no children): Call LLM to select relevant skills from the leaf's skill list
2. **Intermediate node** (has children):
   - If `children ≤ expand_threshold`: Auto-expand all children (no LLM cost)
   - Else: Call LLM to select relevant children (NODE_SELECTION_PROMPT)
3. **Early stop**: If exactly 1 child selected and total skills in subtree ≤ `early_stop_skill_count`, collect all without further recursion
4. **Parallel search**: Multiple selected children searched concurrently via `ThreadPoolExecutor(max_workers=min(len(children), self.max_parallel))`

### LLM Selection

Uses `litellm.completion()` with temperature 0.3, caching enabled, and JSON array output format:

- **Node selection**: `["child_id_1", "child_id_2"]` — picks relevant branches
- **Skill selection**: `["skill_id_1", "skill_id_2"]` — picks relevant skills
- **Response parsing** (`_parse_selection`, line 603): Handles markdown code blocks, regex extraction, and both legacy (string IDs) and new (dict with id+reason) formats

### Pruning (V3 Workflow-Stage Model)

The `SKILL_PRUNE_PROMPT` (prompts.py:188-246) decomposes tasks into:
- **Upstream** (0-2 skills): Gather & prepare — what input/content does the user need first?
- **Production** (1-5 skills): Create & build — what tangible deliverables need to be created?
- **Downstream** (0-2 skills): Deliver & distribute — how does created content reach its audience?

This model deliberately fights keyword-matching: `"promote" ≠ only marketing/SEO skills` — effective promotion requires assets to promote. The prompt inclues explicit anti-pattern warnings.

Output format: `{workflow_analysis, upstream: [{id, role}], production: [{id, role}], downstream: [{id, role}], eliminated: [{id, reason}]}`

### Event System

The Searcher has an `event_callback` mechanism emitting events at every stage: `search_start`, `node_enter`, `children_selected`, `skills_selected`, `early_stop`, `prune_start`, `prune_complete`, `search_complete`. Enables WebUI integration and debugging.

---

## Tree Building (builder.py)

### Two-Phase Build

1. **Root assignment**: LLM assigns all skills to 5 fixed root categories via `FIXED_CATEGORY_ASSIGNMENT_PROMPT`. The 5 categories are hardcoded in schema.py:
   - content-creation, data-processing, development, automation, domain-specific

2. **Recursive splitting**: For each oversized category (> max_skills_per_node), LLM splits into sub-groups using `RECURSIVE_SPLIT_PROMPT`. Continues until all groups ≤ threshold or max_depth reached.

### Queue-Based Parallel Construction

Uses `ThreadPoolExecutor` with `wait(futures, return_when=FIRST_COMPLETED)` (line 182):
- A queue feeds nodes to a thread pool
- When any split completes, its children immediately join the queue
- This maximizes parallelism — no waiting for all siblings to finish before continuing

### Split Quality Validation

`_validate_split_quality()` (line 480) detects:
- Oversized groups (> 2.5× average)
- Singleton groups (1 skill)
- Missing descriptions
- Low coverage (< 90% skills assigned)

### Graceful Degradation

- If LLM grouping fails → node becomes a leaf with all skills (no crash)
- If max depth reached → warning + force leaf
- If unassigned skills remain → assigned to largest child group
- Skills with < 2 members → merged into parent

---

## Plugin Integration (plugin/__init__.py)

### Borrow Mode

LLM credentials cascade through environment variables:
1. `SKILL_RETRIEVER_LLM_API_KEY` → `OPENAI_API_KEY`
2. `SKILL_RETRIEVER_LLM_BASE_URL` → `OPENAI_BASE_URL`
3. `SKILL_RETRIEVER_LLM_MODEL` → `OPENAI_MODEL` → `"gpt-4o"`

Zero additional configuration for users already running Hermes with OpenAI.

### Hook Behavior

`_on_pre_llm_call()`:
1. Check `SKILL_RETRIEVER_DISABLE=1` → return None
2. Skip messages < 10 chars (greetings, follow-ups)
3. Lazy-load Searcher singleton (first call only)
4. Run search, format top-5 as `[Skill Retrieval Hint]` block
5. Return `{"context": hint_block}` — prepended to user message

The LLM sees hints before the user's message and may call `skill_view(name)` to load any suggested skill.

### Safety Badges

Each hint carries source + safety badges:
- `🔒hermes` — installed via Hermes, trusted
- `🌐community` — from AgentSkillOS corpus, unreviewed
- `⚠️` suffix — flagged by safety scan

---

## Dual Scanner Architecture

### Corpus Scanner (`tree/skill_scanner.py`)

`SkillScanner` class reads SKILL.md files from a corpus directory:
- Parses YAML frontmatter for name, description
- Falls back to first paragraph of body if no description
- Loads metadata (github_url, stars, author) from `skills.json`
- Used by TreeBuilder for offline taxonomy construction

### Hermes Scanner (`scanner.py`)

`scan_hermes_skills()` reads the user's local Hermes install:
- Scans `~/.hermes/skills/` and `~/.hermes/hermes-agent/skills/`
- Extracts category from parent directory name
- Parses frontmatter for description and triggers
- Used by the plugin for local skill context

---

## CLI (cli.py)

Four subcommands via argparse:
- `search "query"` — run tree search, print top skills
- `build` — rebuild capability tree from corpus
- `list` — enumerate all skills in corpus
- `info` — system info, tree stats, tree structure summary

---

## Key Design Decisions

1. **LLM-as-classifier instead of embeddings**: The central architectural bet. Vector similarity is narrow; LLM tree navigation applies semantic reasoning at each decision point. This is how the system finds skills that embedding space hides.

2. **Build-once, search-many**: The expensive tree construction (many LLM calls) happens offline. Search uses cheap per-level calls (select N options, output JSON).

3. **Pre-built static tree**: Skills added after build time won't appear. Tradeoffs freshness for speed. The data is gitignored (240MB) and downloaded separately.

4. **Borrow-mode coupling**: Piggybacking on Hermes' LLM credentials eliminates setup friction but ties the retrieval gate to Hermes' model provider.

5. **Fixed root categories**: Hardcoded top-level taxonomy provides stability, at the cost of flexibility for skill types that don't fit the 5 categories.

6. **Workflow-stage pruning**: More sophisticated than typical dedup — the LLM reasons about complete workflows (upstream→production→downstream), not just relevance scoring.

---

## Tests

40 tests across 4 files:

- **test_tree.py** (246 lines): Schema tests (Skill defaults, TreeNode is_leaf/is_intermediate, recursive counting/collecting, serialization, deserialization from both formats), DynamicTreeConfig propagation, TreeBuilder init, builder with minimal skills, frontmatter parsing, scanner with multiple skills
- **test_searcher.py** (129 lines): Searcher init, loading bundled trees, missing tree handling, empty query, event callback lifecycle
- **test_scanner.py** (66 lines): Hermes skill scanner
- **test_plugin.py** (77 lines): Plugin hook behavior

Tests are structural/integration rather than mocking at the boundary — they test against real bundled tree files and create real temp directories for skills.

---

## Comparison with Related Approaches

| Aspect | skill-retriever | Pure Embedding Retrieval | Flat System Prompt |
|--------|----------------|--------------------------|-------------------|
| Discovery | LLM-navigated taxonomy | Vector similarity | Human scans list |
| Context cost | Small per-query | Per-query embedding | Every turn (all skills) |
| Finds non-obvious skills | Yes (semantic reasoning) | Limited (similarity-based) | Depends on human |
| Latency | +1-3 LLM calls | ~1 embedding lookup | 0 (always visible) |
| Scalability | 10K+ skills | Millions | ~200 before bloat |
| Freshness | Rebuild required | Real-time | Real-time |
