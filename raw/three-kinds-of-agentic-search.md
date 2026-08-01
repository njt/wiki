---
url: https://softwaredoug.com/blog/2026/06/08/three-kinds-of-agentic-search
title: "Three Kinds of Agentic Search — Retrieval, Harness, or Model"
author: Doug Turnbull
date_fetched: 2026-08-01
date_published: 2026-06-08
---

# Agentic search – retrieval, harness, or model?

**Author:** Doug Turnbull (SoftwareDoug LLC)
**Published:** June 8th, 2026

## Core thesis

Agentic search gets interesting when agents don't know how to find the right answer. Agents may think they know and might "confidently BS us," but their poor domain intuition steers them astray. They make false assumptions about what *our* users consider relevant — fashionista users interpret "red shoes" as high heels, and at Turnbull's company, ABE meant an A/B testing tool, not a president. Agents need context to know these things, and "context engineering needs agentic search."

The term "agentic search" confusingly refers to three distinct implementation patterns:

1. **Retrieval-centric** – build good search so agents can use it to fill in missing context
2. **Harness-centric** – steer the agent toward needed context, even with bad search
3. **Model-centric** – fine-tune an LLM to know how to search *our* data

---

## Retrieval-centric implementation

When frontier models don't know something, they search — for news, specific technical problems, etc. During training, LLMs see search examples as a technique to learn what they don't know. So the approach is: build good search to solve any conceivable query from an agent.

In an e-commerce catalog example, users searching "red shoes" mean "red high heels," but the agent doesn't know that — luckily it asks search. Initial lexical/vector retrieval pulls back reasonable but naive "red shoe" candidates; the reranker then shapes results toward the intended understanding. Other components may include query understanding, diversity, and custom embedding models.

The key point: "search leads the agent by the nose" toward relevance, overriding the agent's perspective.

### When RAG answers don't look like answers

Most teams build retrieval-centric approaches with classic RAG — chunks of answers and embeddings trained to recognize them. But answers don't always look tied to the question. Example: for "Synopsis of the book Ubik," the answer begins "By the year 1992, humanity has colonized the Moon and psychic powers are common." If you don't know the book, it's unclear this answers the question — the agent says "cool story bro" and ignores the info.

This kind of search is "divorced from the web search trusted by frontier models"; the web contains titles, headings, and other elements that place answers in context.

### The big downside

Search remains hard — "almost nobody builds Google-quality results." And since agents trust search, simple distractors in retrieval can, per Lester Solbakken, "easily confuse reasoning."

---

## Harness-centric implementation

Who should be in charge — should the agent manage the search process, or should search drive the agent? This approach puts the agent in charge, stripping search tools down to core retrieval primitives like a BM25 backend or a filesystem with CLI tools. The agent might struggle more, but it's smart and can figure it out.

However, the agent might find what it *thinks* is relevant, which isn't what's actually relevant. To help, external knowledge is injected: a judge directs the agent, correcting mistakes and guiding it toward better search strategies. When the agent returns results, a judge labels them relevant or not; the agent gets the hint, finding results similar to those labeled relevant and avoiding those labeled irrelevant. Jo Kristian Bergem calls this "relevance feedback on steroids."

### ESCI dataset results (NDCG@10)

| Variant | NDCG@10 | Description |
|---|---|---|
| ESCI BM25 | 0.2895 | Simple BM25 weighing name / description |
| ESCI agentic | 0.4101 | GPT5-mini tool calling loop w/ BM25 + e5 embeddings tools |
| ESCI agentic w/ oracle | 0.5843 | GPT5-mini tool calling loop w/ BM25 + e5 embeddings tools. Judge responds with feedback once |

Trusting the agent's interpretation of queries (row 2) provides a large gain over BM25; injecting domain knowledge via a judge (row 3) improves performance dramatically.

Agents can also be guided ahead of time — teams take inspiration from "skills" in coding agents, using targeted query plans that tell the agent how to use its tools for a specific problem.

### Optimizing content helps harnesses

Content can be optimized for findability — the trick played with coding agents. Instead of relying on dumb chunks, document the purpose of the knowledge. Example: a `/books/scifi/ubik.txt` file with an "About Book Ubik" section noting it's by Philip K. Dick, followed by the synopsis. "Unlike a naive chunk, this bit of information has purpose" — it's clear what problem it solves when in context. This achieves what every search developer wishes content teams would do: "actually optimize content to be findable."

### The downside: cost

Token costs are the major downside. Exploration requires more than issuing a search and retrieving one set of results — it's an agent's exploration of the environment, and that exploration "starts fresh with every new context window."

---

## Model-centric approach

What if the LLM were literally fine-tuned to search efficiently? The harness approach takes a long, winding path to relevant results, wasting calls relearning what works with every fresh context.

A judge helps label some of the agent's reasoning/tool-calling paths as useful and others less so. With labeled traces, an open-weights model can be fine-tuned to proactively make the right tool calls given the input — e.g., knowing exactly how to search "red shoes" after extensive fine-tuning. Retrievers stay simple; the model targets the search task.

Behavior can change just by adjusting the system prompt: "This is an e-commerce search task, optimize for top 10 ranking" versus "You're retrieving chunks for financial research, maximize diversity and relevance at the same time." Models can stay smaller by focusing only on the search part, allowing self-hosting.

### The downside: early days

The OG in the space is SID (sid.ai); contenders like Glean's Waldo model have since entered. They're promising but not widely adopted, with many unknown unknowns. Turnbull suspects tremendous promise ahead: "In other words - stay tuned!"

---

## The alchemy of ambiguity

People use "agentic search" confusingly and inconsistently, focusing on their own part of the problem — a collection of solutions like enriched base metals, with unclear alchemy bringing them together.

Converging trends include: easier-to-organize data (like PageIndex); harness design patterns mirroring coding agents (hooks/evaluators that steer agents, or "skills" guiding them a priori); and, as with coding, "the agents eat the harnesses" — when successful patterns emerge, agentic search models train to memorize them. Retrieval is moving toward agent-centric approaches, with agents having "their own specific, unique retrieval patterns"; late interaction and learned sparse retrieval are having their day via companies like LightOn and MixedBread.

"We're swimming in an interesting retrieval primordial goop" — what emerges may be an obvious combination or look nothing like the component parts. Exciting times.
