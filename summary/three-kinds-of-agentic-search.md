---
url: https://softwaredoug.com/blog/2026/06/08/three-kinds-of-agentic-search
title: "Three Kinds of Agentic Search — Retrieval, Harness, or Model"
author: Doug Turnbull
date_fetched: 2026-08-01
date_published: 2026-06-08
topics:
  - agent-memory-and-context
---

Doug Turnbull argues that "agentic search" is a confusing term that actually
refers to three distinct implementation patterns. All three address the same
core problem: agents don't know how to find the right answer in your domain —
they make false assumptions about what your users consider relevant, and they
need domain context to steer them correctly.

**Retrieval-centric** puts search in charge. Build high-quality search that
solves any conceivable query from an agent, letting search "lead the agent by
the nose" toward relevance. The big downside: search is hard, almost nobody
builds Google-quality results, and agents trust search — so even simple
distractors in retrieval can confuse reasoning. Classic RAG also struggles when
answers don't look like answers (e.g., a book synopsis that opens with plot,
not the title).

**Harness-centric** puts the agent in charge. Give it stripped-down retrieval
primitives (BM25, filesystem tools) and let it explore. Inject domain knowledge
via a judge that labels results as relevant or not — "relevance feedback on
steroids." On the ESCI dataset, agentic search with oracle feedback achieved
NDCG@10 of 0.5843 versus 0.2895 for plain BM25. The approach also benefits from
optimizing content for findability (documenting *why* a chunk exists, like
coding-agent docs do). The major cost: token usage from repeated exploration
that "starts fresh with every new context window."

**Model-centric** fine-tunes an open-weights model to make the right tool calls
for your domain, internalizing what the harness approach learns expensively from
scratch each time. A judge labels tool-calling traces as useful or not; the
model is then fine-tuned on those traces. Behavior can shift via system prompts
("e-commerce ranking" vs. "financial research diversity"). SID (sid.ai) and
Glean's Waldo are early entrants; the approach shows promise but is still in
early days.

Turnbull closes by noting converging trends: easier data organization, harness
design patterns borrowed from coding agents, and a trajectory where "agents eat
the harnesses" — successful patterns get baked into fine-tuned models. The
space is a "retrieval primordial goop" whose eventual shape may look nothing
like its component parts.
