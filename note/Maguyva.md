# Maguyva

Maguyva is a remote MCP server that gives AI coding agents a pre-built, graph-ranked "map" of your codebase — 279 languages via Tree-sitter AST queries, five fused search modalities, and four graph views (Dependency, Type, Data-flow, Control-flow) — so agents query before they edit instead of hallucinating from training-data memory. Built by two humans and 33 AI agents, zero VC.

---

## What It Is

Maguyva is an MCP server you point at your GitHub repos. It does all the heavy lifting in the cloud — indexing, parsing, ranking — and exposes 11 MCP tools that any coding agent can query. The pitch: **"deterministic software — same question, same answer — not another AI."**

The product sits at the intersection of three trends I've been tracking: context engineering as the real bottleneck ([[Context Engineering at the Frontier (Linus Lee)]]), the codebase itself as the AI's primary context document ([[AI Slop Starts with the Codebase Itself]]), and MCP as the standard agent-to-tool integration layer ([[Building Agents for Production Systems with MCP]]).

## How It Works

1. **Connect GitHub** — authorize, pick repos
2. **Cloud builds the map** — Tree-sitter parses every file, builds a ranked symbol graph with dependency/type/data-flow/control-flow edges (2–15 min initial sync)
3. **Plug in any MCP client** — drop API key into your agent's MCP config
4. **Agent queries first** — before touching code, the agent asks Maguyva "where is X?", "what depends on Y?", "what's the blast radius of changing Z?"

The indexing auto-syncs on push to GitHub with incremental parsing — only changed files get re-parsed. Paid plans rebuild the full graph hourly; free tier does daily syncs.

## Key Differentiators

### Graph-Ranked Symbol Importance

This is the detail I find most interesting. Every symbol gets a PageRank-style importance score. A `main()` implementation might score 3,400; a test stub scores 49. Agents don't just find code — they find the *important* code. This is a genuinely useful ranking signal that grep, `rg`, and even most LSP servers don't provide.

### Five Search Modalities, Fused

Semantic search (embeddings), structural search (AST-aware), graph traversal (dependency chains), plain text retrieval, and ranked fusion across all four. This matters because coding agents have wildly different query patterns — sometimes they need "find the function that processes payments," sometimes "show me everything that calls this function." One search modality can't serve both.

### Remote, Not Local

The entire pipeline runs in the cloud. Your laptop doesn't index anything, doesn't run Tree-sitter, doesn't build embeddings. This is the opposite of local-first tools — and it's the right call for teams. A new developer (human or agent) shouldn't need to build the index before they can ask a question. See [[The GUS Stack — Go, Unix, SQLite]] for the counter-argument that local-first is better for agents, but Maguyva's cloud-first approach is the pragmatic team play.

> "Not one regex pretending it understands Haskell"

Tree-sitter AST queries across 279 languages. This is the right technical choice. Regex-based code search is the "vibe coding" of code understanding — it works until it doesn't, and when it doesn't, it fails silently. See [[Guardrails and Feedback Loops]] for the broader principle: deterministic enforcement beats probabilistic guessing.

> "The cost scales with how much code you index — not how many humans or agents query it."

Per-workspace pricing, not per-seat. This is a pricing model designed for the agent era — when a single developer might have 5 agents querying the codebase simultaneously, per-user pricing would be absurd. The company explicitly calls this out: rejecting "value-capture driven pricing from an era when every user had a pulse."

> "I preferred when I could just make up function names and nobody noticed." — GPT-4o (simulated review, 3/5)

The simulated negative review is a knowing joke, but it captures something real: agents without codebase context compensate by inventing APIs. Maguyva's thesis is that deterministic lookup beats creative hallucination.

## The Meta Angle

Maguyva is built by a fleet of 33 AI agents — two humans plus AI. They're dogfooding their own product: the agents building Maguyva use Maguyva to understand the Maguyva codebase. The 30.9K+ commits and 4.6M+ lines of code indexed are partly their own.

"Zero budget meetings" and "No venture capital was consumed" reads as positioning against the overfunded AI startup narrative. Whether true or marketing, it's a coherent stance: two people, a fleet of agents, and a deterministic product.

## Critical Analysis

**What Maguyva gets right.** The graph-ranked importance signal is genuinely novel and useful. Most code search tools return results ranked by text relevance or recency; neither signal captures *structural* importance. A function that's called from 50 places is more important than one called from 2, and Maguyva encodes that. The five-modality fusion is also a real differentiator — most tools pick one search strategy and optimize it; Maguyva bets that fusion beats specialization.

**What's underspecified.** The page is light on what happens when the graph is wrong. Tree-sitter is fast and reliable for syntax, but cross-file dependency resolution — especially in dynamic languages like Python, Ruby, or JavaScript — requires more than AST parsing. The page claims 279 languages but the depth of support for dynamic dispatch, metaprogramming, or runtime reflection is unclear. A 3,400-ranked `main()` is easy; correctly ranking a Rails controller method that's called via convention-over-configuration routing is hard.

**The remote bet.** Cloud-only indexing means Maguyva knows your entire codebase graph. For enterprises, this is a data-exfiltration concern that the page doesn't address. The privacy and security pages are linked but the homepage doesn't surface the architecture for handling proprietary code. Compare with [[cco]], which sandboxes agents locally — Maguyva takes the opposite approach.

**The competitive landscape.** Maguyva competes with LSP-based tools (most IDEs already have go-to-definition and find-references), grep/`rg` (free, fast, shallow), and the agent's own training-data memory (free, hallucination-prone). The value proposition is that Maguyva is more structured than grep, more comprehensive than LSP (graph views beyond call hierarchy), and more grounded than training memory. Whether this delta is worth paying for depends on how much your agents hallucinate without it — and that depends on how conventional your codebase is (see [[AI Slop Starts with the Codebase Itself]]).

**The pricing model is a quiet innovation.** Per-workspace pricing aligned with indexing volume, not per-seat, is the right model for agent-heavy teams. It also aligns incentives: Maguyva's revenue grows with your codebase size, not your headcount. This is pricing designed for an era where code velocity outpaces team growth.

---

*Sources: [[raw/maguyva]]*
*Last updated: 2026-07-21*
