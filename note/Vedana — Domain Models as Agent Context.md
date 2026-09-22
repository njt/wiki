# Vedana — Domain Models as Agent Context

Olga Tataranova (co-founder of Epoch8) presents Vedana, a system for making LLMs answer domain-specific questions *correctly* rather than plausibly. The bet, in one sentence: don't dump documents into the model — build a hand-crafted domain model (anchors, attributes, links) and hand *that* to the agent as its context and its query grammar. This page distills the argument, the quotes, and where the approach is genuinely new versus where it oversells.

---

## The Argument

LLMs fail on domain questions because they lack business logic. "ChatGPT is great in generating some good prose, some good texts. But ChatGPT doesn't know your domain, doesn't know how your business processes work, doesn't know anything about what you do." For Epoch8's customers — legal, e-commerce, manufacturing, automotive — a wrong answer is expensive, and it has to be traceable, which is why "sounds right" is not good enough.

The fix is a structured domain model, not better chunking. Epoch8 uses Alexey Mahotkin's **minimal modeling** notation: *anchors* (the nouns), *attributes*, and *links* (the relationships). The model is maintained by domain experts in [[Grist]], fed to LLMs in two ways — as a prompt to extract entities from documents, and as a context file (the whole Grist SQLite database, or via Grist MCP) so the agent can explore the schema and write Cypher queries against Memgraph, the production graph database.

## Key Quotes

> "We stopped just sounding right and we started being right."

The whole talk in eight words. The concrete comparison is a case where ChatGPT returned an answer in USD and Vedana returned the correct value by fetching it from the database. "Being right" here is *operational* — it means an exact value plus the source, not a better probability distribution.

> "Links are the most important part of the description because links actually describe how your domain works, how the agent reasons across your domain."

This is the most important sentence in the talk, and it's understated. Entities alone (a judge has a name, a title) don't encode reasoning; relationships do (a law document *amends* another, a court decision *has parties*). It is the typed-edge thesis from [[Context Graphs]] applied to domain knowledge rather than decision memory: the structure you need is *edges*, not embeddings.

> "We just dumped this SQLite database to Claude or to any other AI agent and it is comfortable enough to explore the data model, to explore the tables."

The surprisingly simple mechanism at the center of it: the entire Grist document — a SQLite file — becomes context. No elaborate RAG pipeline. The schema *is* the interface.

> "When MCP arrived it was like the game changer because we just fed the Grist MCP to Claude and we were like, 'Dear Claude, we need to set up an AI assistant for our customer. Please go and check this data model. Please go check data for consistency and write some prompts.'"

A nice inversion of the usual tool-calling story: here the frontier model's job is not to answer the end user, but to *bootstrap* a cheaper, smaller model by reading the domain model and writing its prompts. Claude as model-of-record authoring the playbook the cheaper LLM then executes.

> "Expertise is extremely hard to scale. Very hard to scale. And no AI can help you here."

The blunt admission that undercuts the demo. The domain model itself "can't be generated" — Claude "misses the point, misses the links, misses the domain logic." The hard part stays human, and it doesn't scale.

## Key Themes

- **#concept** — The domain model as the reasoning backbone: structure the *domain*, not the chunks.
- **#concept** — Links (typed relationships) as the carrier of business logic; entities alone are inert.
- **#pattern** — Two-layer data strategy: structured extraction primary, raw text chunks as a quality-degrading fallback.
- **#tool** — Grist as an expert-editable, typed, single-source-of-truth layer whose SQLite file doubles as LLM context; Memgraph as the production query backend via Cypher.
- **#pattern** — Using a frontier model (Claude) to bootstrap cheaper LLMs by having it read the model and write the prompts.

## Critical Analysis

**The core insight is real and convergent.** "Feed the agent structured domain knowledge, not raw documents" is the same conclusion the wiki keeps reaching from different directions — [[Context Graphs]]' typed edges, [[Domain Storytelling]]'s machine-readable sentence diagrams, [[Cerebras Knowledge Base Architecture]]'s structure-at-ingestion-time. Vedana's contribution is a concrete production shape: a hand-built model, a query grammar baked in, and a two-layer fallback. That's more specific than most context-engineering essays get.

**But the ChatGPT comparison is a straw man.** They chunked documents and dumped them into ChatGPT, then declare victory. That's the baseline the whole field left behind in 2023. A well-tuned RAG pipeline with hybrid search, reranking, and the same structured metadata isn't in the comparison, so the claim "domain models beat RAG" is unproven — the talk only shows domain models beat *naive chunking*.

**The hard questions are all unanswered.** How does the agent "explore" the SQLite file — is there a RAG step, or does it read raw bytes? What stops the on-the-fly Cypher queries from being destructive or slow on a production graph database? Where's the evaluation beyond one anecdote? The talk is a demo, not evidence, and it doesn't pretend otherwise — but the omission of guardrails and metrics matters precisely because the customers are legal and manufacturing, where a wrong answer is expensive.

**The cleanest tension is with [[DDD Matters More When AI Writes Your Code]].** Smółka argues the value of a domain model is the *team's understanding*, not an artifact — "an AI-generated wall of text doesn't help you figure out the business problem." Vedana treats the model as an artifact you hand to the agent. Yet the two agree on the one thing that matters most: the model can't be auto-generated. Both sides converge on "expertise doesn't scale, and no AI fixes that," which is quietly the most important sentence in either piece.

## Related Pages

- [[Grist]] — strengthens it by showing the tool's real value isn't its formula engine but its *shape*: typed columns and a single SQLite file make it a domain-model authoring surface that doubles as agent context.
- [[Context Graphs]] — nuances it: Vedana's anchors/attributes/links are typed edges aimed at *domain knowledge* rather than decision rationale, confirming that "similarity is not relevance" generalizes beyond memory into modeling itself.
- [[Domain Storytelling]] — complicates it from the sibling method: both produce a structured domain model, but where Domain Storytelling optimizes for *human* shared understanding, minimal modeling optimizes for *machine* readability — the two grammars for the same need.
- [[DDD Matters More When AI Writes Your Code]] — the sharpest point of friction: Smółka says the model's value is understanding, not artifact; Vedana makes the model the artifact the agent reads. They reconcile on the one claim both refuse to concede — the model can't be generated by the LLM.

---
*Sources: [[raw/vedana-domain-models-as-agent-context]], [[summary/vedana-domain-models-as-agent-context]]*
*Last updated: 2026-09-13*
