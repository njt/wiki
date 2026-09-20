---
url: https://www.oreilly.com/radar/when-smaller-models-win/
title: "When Smaller Models Win"
date_fetched: 2026-09-20
topics:
  - local-and-open-source-inference
  - ai-research-and-models
---

An O'Reilly Radar essay (republished from the Asimov's Addendum blog) arguing that the best tool for a task is often not the most general model. Its anchor example is chess: LLMs trained on supercomputers famously cheat at chess, while Stockfish — a small, specialized tree-search engine running on consumer hardware since 2008 — has beaten grandmasters for a decade. The lesson: raw general intelligence does not automatically translate into specialized capability, and the opposite is often closer to true.

The essay surveys recent evidence that small specialized models beat frontier general ones at narrow tasks: a 4B search-research model outperforming Claude Sonnet 4.5 on some benchmarks, a 4B terminal-execution model that larger agents hand work off to, 258M-parameter PDF extraction models, and a 4B negotiation model beating GPT-5-class systems. It cites an NVIDIA position paper defining small models as under 10B parameters and arguing they are the future of agentic AI.

Practically, it walks through LoRA fine-tuning (cheap, portable, resistant to catastrophic forgetting), hosting options, and the deeper argument for self-hosting: control over the inference pipeline itself — constrained outputs, token probabilities, caching, task-specific logic — that no chat-completion API offers. Convenience favors generic models early; specialization wins as the problem narrows.
