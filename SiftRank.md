# SiftRank

A Go tool implementing the SiftRank algorithm for ranking any dataset by relevance to a natural-language prompt. Uses LLMs for pairwise comparisons within random batches, detects inflection points to separate relevant from irrelevant items, and iterates until scores stabilize. Produces deterministic rankings in seconds for modest datasets, costing pennies.

---

## Key Themes

#information-retrieval #ranking #llm-tools #search #go

The algorithm is cleverly designed around LLM limitations: stochastic sampling handles the context window constraint, fixed-complexity caps prevent cost explosions, and iterative refinement compensates for LLM nondeterminism. The result is deterministic rankings from a nondeterministic oracle -- a neat trick.

This is useful infrastructure for the [[LLM Wiki]] pattern: when you have hundreds of raw sources and need to find the most relevant ones for a given topic, SiftRank could automate the prioritization step. Also connects to [[LLM Evals]] -- ranking outputs by quality is a form of evaluation.

## Critical Analysis

Strong: the design constraints (linear scaling, deterministic output, pennies per run) make this practical rather than theoretical. Written in Go with no dependencies beyond OpenAI API calls, so it's easy to integrate into pipelines. The elbow method for finding natural score breakpoints is a good heuristic.

Weak: the algorithm assumes relevance is a single dimension, which works for simple queries ("find things about time") but breaks down for multifaceted research questions. No support for explaining why items ranked where they did -- you get scores but not reasoning. The reliance on OpenAI-compatible APIs means you're paying per comparison, and costs scale with dataset size even if they're low per item.

For the use case of "I have 100 bookmarks and want to find the 10 most relevant to topic X," this is ideal. For more nuanced curation, you'd want something that preserves the LLM's reasoning about relevance.

---
*Sources: [[raw/siftrank]]*
*Last updated: 2026-05-14*
