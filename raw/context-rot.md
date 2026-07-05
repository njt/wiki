---
url: https://roampal.ai/blog-context-rot.html
date_fetched: 2026-07-05
backfilled: true
---

Chroma's research confirmed something I'd been hitting head-on: bigger context windows don't solve memory, they make it worse. They tested a collection of models and found that as you stuff more conversation history into a prompt, the model's accuracy degrades, losing context, producing worse answers.

The real issue goes deeper: retrieval and generation are decoupled. The retriever finds "relevant" chunks, the generator uses them, but nothing connects what got retrieved to whether the answer actually helped. Rerankers, query rewriting, hybrid search. They all try to fix retrieval quality at query time. They optimize for similarity, and do it well, but what about success?

Your AI pulls up a memory, uses it, gets it wrong. That memory sits there waiting to surface again with the same confidence. What if it had a feedback loop? I built outcome-based learning: when you say "that worked" or "no, that's wrong," those signals attach to the memories themselves. Good ones surface more. Bad ones sink.

## How Memories Are Created and Retrieved

If the user has to click a thumbs up button, they won't. A sidecar LLM handles it instead: after each exchange it infers whether the response helped or not, summarizes the exchange, extracts facts from it, applies tags, and scores memories. Every function is compression. Summarizing replaces raw conversation history with distilled context (roughly 300 characters), fact extraction pulls hard specifics out of noise, tagging enables tag-first cascade lookup instead of blind vector search.

Fresh memories enter at working tier (24h) and move through history (30d) to patterns (permanent) as they accumulate positive feedback. Scores accumulate through flat deltas: worked adds +0.2, failed subtracts -0.3, partial adds +0.05, unknown subtracts -0.05. Only patterns are protected from decay, and even then they can demote back into an unprotected tier if they prove unhelpful over time.

### Why extract tags?

The sidecar extracts noun tags from every memory and adds them to an in-memory known-tag index. At query time, whether triggered by a user prompt or the LLM itself manually searching its own memory, word-boundary matching finds which of those tags appear in the query text. For each matched tag, ChromaDB returns candidates that carry it. Results are counted by overlap, how many of the query's tags each memory shares, and cascade from highest overlap down to single-tag hits. Remaining slots fill with semantic similarity before a cross-encoder reranks the final pool for precision.

Two lanes retrieve separately, with summaries carrying narrative context about the exchange while facts capture hard specifics. Each retrieved memory surfaces with outcome metadata (score, Wilson confidence, use count, recent outcomes) so the LLM sees not just what matched but how reliable each one has proven to be.

## The Benchmark

I tested Roampal on a corrected LoCoMo benchmark: the LLM recorded memories over long multi-session conversations, then was tested on its recall across 1,986 questions graded end-to-end.

| Approach | Answer Accuracy | vs Raw Ingestion | 
|---|---|---|
| Raw ingestion (standard RAG) | 53.0% | — | 
| TagCascade (Roampal) | 76.6% | +23.6 pts | 

+23 points over raw ingestion (p<0.0001). The system learned entirely through conversation across 10 conversations with 20 characters sharing 3,015 facts. Out of that dialogue came approximately 5,000 memories. TagCascade proved surgical at scale.

## The Answer

Chroma's research showed that stuffing more conversation history into a prompt makes accuracy worse. The answer isn't bigger context windows, it is knowing what to leave out. I built Roampal with a sliding window of four exchanges plus surgical retrieval through TagCascade. Instead of drowning the LLM in everything ever said, it pulls only the memories that actually helped before, and those memories earn their place through outcome feedback, promoting over time or decaying when they don't.

Bigger context windows will keep getting bigger. Retrieval will keep getting fancier. And none of it matters if the system cannot learn from whether the answer helped.

**Want to learn more?** GitHub | Benchmarks

**OpenCode:**

```
pip install roampal-core
roampal init --opencode
```
Uses a sidecar LLM that handles extraction, summarization, and tagging automatically.

**Claude Code:**

```
pip install roampal-core
roampal init --claude-code
```
Uses an MCP tool where the main LLM manages extraction, summarization, and tagging.

**Want it all in one app?** Get Roampal Desktop

Your memories stay local. Your learning stays yours.
