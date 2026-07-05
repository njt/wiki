# Layer-First Pattern — Keep Data Out of the LLM Context

When tool calls pass large datasets through an LLM that doesn't need to reason about them, the model becomes an expensive, lossy data pipe. The layer-first pattern keeps data server-side and returns lightweight acknowledgments instead — reducing context from 125K+ tokens to ~150 tokens per interaction while making the render pipeline deterministic and independently testable.

---

## Key Quotes

> "If the LLM isn't making a decision based on the content, it shouldn't be holding the content at all."

This is the organizing principle. It is not "context windows are too small" — it is that we are using the LLM as a data transport layer when it should be a judgment layer. The distinction echoes [[Smart Models Dumb Pipes]]'s core thesis: models own decisions, infrastructure owns execution.

> A modest wildfire dataset in GeoJSON can be "50–500KB of raw GeoJSON," which at ~4 bytes per token translates to roughly 125,000 tokens — exceeding many context windows and creating high costs.

The numbers make the case. This is not a theoretical concern — it is a concrete cost and performance wall that any real-world tool-calling system hits immediately. The 4 bytes/token heuristic is a useful rule of thumb for spotting these problems before they manifest.

> "Tool A fetches data → LLM receives it → LLM passes it directly to Tool B" — the LLM is misused as a data pipe.

This is Bogh's diagnostic pattern. If the data flows through the LLM without the LLM making a decision about it, you have a composition problem. The fix is not better prompting; it is moving the composition out of the context window entirely.

---

## Key Themes

#pattern #tool-design #context-engineering #architecture

**The Layer Stack as Composition Model.** The implementation is elegantly simple: an in-process `Map<sessionId, layer[]>` with a 30-minute TTL. Each tool call appends to the array; the render step composites in insertion order. This is Mapbox's layer model replicated server-side — the LLM orchestrates which layers to fetch and in what order, but never touches the geometry. The renderer is swappable (static tiles → headless Mapbox GL JS in Playwright) without interface changes, which is the real test of a clean abstraction.

**Deterministic Rendering, Nondeterministic Orchestration.** The split is clean: the LLM decides *what* to show (which layers, what order), and deterministic code decides *how* to render it. This is the same boundary [[Tone LLM]] draws — LLM fills a small JSON schema, deterministic code translates to plugin config. The render pipeline is independently testable because it has no model dependency. The LLM's decisions are testable because they're small enough to inspect manually.

**Where the LLM Goes Blind.** The explicit tradeoff: the LLM "can't reason about the underlying geometry" of queued layers. For a map compositor this is fine — the user sees the rendered map and can iterate. But for analytical tools where the LLM *should* reason about data content, this pattern would be destructive. The boundary is whether the LLM needs to make content-level decisions or only coordination-level decisions. This aligns with [[Elements of Agentic Systems Design]]'s Context element — the model only sees what it needs for the decision at hand.

**Ephemeral State as Deliberate Design Choice.** The 30-minute TTL on layer queues is not a limitation to be fixed — it is a memory-is-ephemeral architectural constraint that prevents stale state accumulation. Bogh notes you could persist layers to a database for multi-turn refinement, but the default of ephemeral state forces clean interaction design. This is the same tension [[Agent Memory and Context]] documents across implementations: what to remember, what to forget, and how to know the difference.

---

## Critical Analysis

The pattern is sound and broadly applicable, but the article understates two challenges.

**First**, the pattern shifts complexity from the LLM to the composition layer. For maps, the composition is straightforward (z-order overlay). For the general case — multi-source data enrichment, log analysis — the compositor becomes a domain-specific merge engine. That engine has its own correctness surface, its own test burden, and its own maintenance cost. The article presents this as a win (and it usually is), but it's not cost-free. You are trading LLM context cost for compositor development cost, and the breakeven depends on how many times the compositor gets reused.

**Second**, the pattern assumes the LLM doesn't need to see intermediate data to make good decisions. For map layers this holds. For the log analysis example Bogh cites, it's less clear — sometimes the *distribution* of log entries matters to the next analytical step, not just the final answer. A compositor that summarizes into a chart hides the distribution; one that returns raw counts leaks it. Designing the acknowledgment payload ("what does the LLM actually need to know to decide the next step?") is the hard design work, and the article doesn't give much guidance on it.

**What the article gets right** is the diagnostic test: if data passes through the LLM without the LLM making a decision about it, the architecture is wrong. That single rule catches an enormous amount of real-world tool design waste. It is the same instinct behind [[How Hightouch Built Their Long-Running Agent Harness]]'s "scratch paper" pattern — spawn subagents for messy work, return only summaries — and [[Maybe Coding Agents Don't Need a Bigger Memory]]'s argument that context ≠ continuity. The common thread: the LLM is a judgment engine, not a data warehouse.

Bogh's framing of tool design as *shaping* what the LLM sees ("the range of reasonable responses all lead to correct outcomes") is the right ambition. It moves tool design from "expose capability" to "constrain toward correctness," which is where [[Guardrails and Feedback Loops]] lands from the other direction — deterministic enforcement, not instructions.

---

*Sources: [[summary/mapbox-llm-composition]]*
*Last updated: 2026-07-05*
