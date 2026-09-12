# Elysia

Weaviate's open-source agentic framework that structures tool selection as a decision tree. Instead of dumping all tools into a flat context window, each node has a "decision agent" with global awareness that routes to the appropriate next action. Ships as both a FastAPI app and a Python library, with DSPy handling LLM interactions and first-class Weaviate vector DB integration. Currently beta (v0.2.8).

---

## Architecture: Decision Tree Over Flat Tool List

> "Each node in the tree is orchestrated by a decision agent with global context awareness about its environment and its available options."

This is the core idea and it's genuinely interesting. Most frameworks give an agent a flat list of tools and let the LLM sort it out via function calling. Elysia structures the decision *before* the LLM sees it by routing through a tree. Each node is a constrained choice, not a haystack search.

The trade-off: a decision tree is rigid. Adding a new tool means finding the right place in the tree, not just appending to a list. This is both the strength (constraint reduces error) and the weakness (overhead of tree design).

## Weaviate Integration as First-Class

> Built-in query and aggregate tools for interacting with Weaviate collections.

The framework's built-in tools are domain-specific (vector search, aggregation), not generic (web search, calculator). This is a framework for data-intensive agents, not general-purpose chatbots. The `preprocess()` step that analyzes collection schemas suggests the tree is partially auto-generated from data shape, not purely hand-crafted.

## DSPy Under the Hood

Using DSPy rather than raw LLM calls or LangChain is a pragmatic choice. DSPy's compiler approach (optimize prompts, not hand-write them) fits the decision tree metaphor: each node is a constrained optimization problem. But it also means Elysia inherits DSPy's complexity and learning curve.

## App + Library = Onboarding Gradient

The dual-mode design (`elysia start` for a web UI, or `import elysia` for programmatic use) is smart. Non-engineers get a settings page and chat UI; engineers get full control. The companion frontend repo ([elysia-frontend](https://github.com/weaviate/elysia-frontend)) suggests the app is where Weaviate sees the product value.

---

## Critical Analysis

**What's novel:** The decision tree as constraint mechanism. Most agent research focuses on giving models *more* tools and better reasoning. Elysia goes the opposite direction: restrict choice at each step so the model can't fail by picking wrong. This aligns with [[Harness Engineering]]'s feedforward control — constrain the action space rather than pleading with the model.

**What's concerning:** The tree must be designed. For Weaviate's use case (structured data queries), the tree probably writes itself — you know the collections, you know the query types. For open-ended use cases, tree design becomes prompt engineering by another name. The readme's call for community contributions to "create new tools" sidesteps the harder problem: where in the tree do new tools belong?

**The Weaviate lock-in question:** BSD-3-Clause and the company calls it a community project. But the built-in tools are Weaviate-specific. Using Elysia with Pinecone or pgvector means writing your own tool set. The framework is open-core in spirit if not in license: the tree runtime is open, but the valuable tools point at Weaviate. This is [[Smart Models Dumb Pipes]] applied to business model: the tree is a dumb pipe; the Weaviate tools are where the money flows.

**Beta reality:** v0.2.8, 14 open issues, explicit beta warning. The architecture is more interesting than the current implementation. Worth watching for the decision tree pattern, not for production use today. The decision-tree approach to tool selection may influence frameworks that don't carry Weaviate-specific baggage.

**The deeper pattern:** A decision tree is a hand-crafted version of what [[Slate]] does with context routing and what [[Scaling Long-Running Agents]] found with planner/worker/judge. The same insight keeps appearing: flat tool access doesn't scale. Structure the decision space, don't just expand it.

---

*Sources: [[summary/elysia]]*
*Last updated: 2026-05-14*
