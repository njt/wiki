---
url: https://www.youtube.com/watch?v=JMA9D8J9EyI
gist_url: https://gist.github.com/9e94fbdabe3777afd51dfea384f5c37f
title: "Everyone wants bigger context windows. Linus Lee thinks that's the wrong instinct"
author: Linus Lee (Head of AI, Thrive Capital)
date_fetched: 2026-07-04
date_published: 2026-05-14
channel: AI Council
duration: 16m 48s
ytx_by: njt
topics:
  - agent-memory-and-context
---

# Everyone wants bigger context windows. Linus Lee thinks that's the wrong instinct

Linus Lee (Head of AI at Thrive Capital, formerly Notion) previews his AI Council talk "Context Engineering at the Frontier." His core argument: bigger context windows are a brute-force crutch, and the real design work is in composable retrieval pipelines, write-time indexing strategies, and semantic observability.

## Key points

- **Context windows are a brute-force crutch.** Throwing everything into a monolithic context is a fine baseline, but it's expensive, slow, and brittle. The real design work is deciding what *not* to put in context and how to structure the retrieval pipeline that feeds it.
- **Composability beats monoliths for engineering velocity.** A single agent that does everything in one shot means changing any part forces you to re‑run all evals. Breaking the system into retrieval sub‑agents or tools with defined interfaces lets you iterate on each piece independently without destabilizing the whole.
- **Retrieval is a pipeline, not a single tool call.** The path from raw data to the final model's context is a gradient of techniques: lightweight re‑rankers, structured‑query writers, full sub‑agents. You can push work down into cheaper, more specialized components and reserve the expensive top‑level context tokens for synthesis.
- **Context engineering is a search problem in disguise.** Classic information retrieval ideas—iterative refinement, over‑retrieving for recall then narrowing for precision, merging parallel retrieval methods—map directly onto how you should build agent context. The 1968 IBM comic on IR still captures the essence: structure information so the computer can understand it, balancing precision and recall.
- **Indexing strategy unlocks new query capabilities.** How you ingest and pre‑structure data (at write time) determines what questions you can later ask. Sparse vectors from mechanistic interpretability (e.g., sparse autoencoders) could give dimensions that are human‑legible, enabling queries that dense embeddings can't support.
- **Scale erodes intuition; semantic observability is the missing piece.** As token volumes grow, you lose the "hand feel" for where models stumble. The harder problem isn't infrastructure health—it's knowing whether runs are *semantically* successful (did the user actually get a deck, or did the model silently fail?).

## Key quotes

- **On the hidden cost of monolithic context:**
  "If you have a single agent doing a single monolithic inference… and then you want to change some part of it… you have to run all of the evals again of anything that touches this particular generation step because it's one monolithic system."
- **On the under‑appreciated benefit of retrieval sub‑agents:**
  "Not only is it saving costs and potentially saving latency and improving performance, but it also breaks the total system down into composable sub pieces where… you can iterate on just the retrieval subagent and make it really, really good. And then that doesn't impact the rest of how the top level agent interacts with all of his other tools."
- **On the real design space for context:**
  "The design space is less 'just what do we put in the context and how long is the context?' and more 'okay, we can spend the expensive tokens in the final model's context… or we can push some of that retrieval work down into more and more specific tools that are less and less costly, but require maybe a little more design thinking.'"
- **On reframing the whole problem:**
  "Context engineering as a search problem… You define some space of options… and then you have some way of either narrowing down your search base or enumerating the space so that you can iteratively search through it."
- **On the observability gap at scale:**
  "Are these runs or are these pipelines delivering results in a way that is sort of semantically successful in the way that the user would define it? Like when they want to generate a deck, is there a deck at the end of it? Or is the model kind of stumbling through and not delivering a text answer instead?"

## Tools, practices, and methodologies

- **Retrieval sub‑agents** — Composable, independently iterable agents that handle a specific retrieval task and return a curated subset to the top‑level agent. They break a monolithic system into pieces with defined interfaces, allowing optimization of one sub‑agent without invalidating the whole system's evals.
- **Iterative refinement pipeline** — A multi‑stage retrieval process starting with fast, high‑recall over‑retrieval, then progressively narrowing results using re‑rankers, sub‑agents, or parallel methods merged together. Apply the classic search stack pattern: optimize for recall early, precision late.
- **Write‑time pre‑structuring** — Massaging and structuring data at ingestion time—extracting semantic units, building indexes, or encoding into specialized representations—so that queries at read time can be more expressive. Shift work to indexing time to enable questions impossible with a raw page index.
- **Sparse vectors from interpretability (conceptual)** — Using sparse autoencoders or similar techniques to produce embeddings where each dimension corresponds to a legible concept, rather than opaque dense vectors. Index data with interpretable sparse vectors to target specific semantic dimensions.
- **Semantic observability** — Monitoring not just whether a pipeline completed without errors, but whether the output was *semantically* what the user intended. Build tooling that recovers a "hand feel" for where models struggle and what percentage of runs deliver successful outcomes.

## Unanswered questions and omissions

- How do you decide the write‑time vs. read‑time split in practice? No heuristics or decision framework is offered.
- Sparse interpretable vectors are entirely speculative — no concrete implementation, benchmarks, or sketch of how they'd be built or evaluated.
- No engagement with the counter‑argument that bigger context windows may be the simpler, good‑enough solution, especially as they get cheaper and models improve at long‑context reasoning.
- No discussion of how stochastic generation, prompt sensitivity, or the non‑deterministic nature of LLM tool use complicate the precision/recall framing.
- Semantic observability is named as a problem, but no solution is offered — what would such tooling actually look like?
- The talk is silent on evaluation — if you decompose a monolith into sub‑agents, how do you evaluate each piece in isolation?
- No discussion of data freshness, permissions, or security in retrieval pipelines.
