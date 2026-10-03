# From Raw Data to Graph-Native AI

An O'Reilly Radar essay arguing that the graph-in-AI conversation starts too late: we debate GNNs, GraphRAG, and graph foundation models while skipping the step that determines what all of them can do — deciding what the nodes and edges should be. The author proposes "graph-native AI": graph modeling promoted to a first-class, evaluated stage between raw data and every downstream consumer.

---

## The Argument

The essay's framing device is a gap: "Most of the conversation starts after the graph already exists." Everything downstream — embeddings, neighborhoods, community summaries, agent memory — inherits the quality of choices nobody treats as a discipline: what counts as an entity, what a relationship means, how time is represented, which source wins when records disagree. A customer can be one node or a set of legal entities changing over time; a purchase can be a relationship or a timestamped event; duplicate records can be merged, linked, or left apart. There is no single answer — only answers relative to the task.

The sharpest section takes on an assumption hiding in relational deep learning: that a database schema designed for storage and transactions is also a useful graph for machine learning. Recent work across 26 relational tasks found schema-derived graphs suffer "information overload and semantic fragmentation," and that pruning edges and adding dependencies the schema never captured improves performance. Graph construction, in other words, needs its own evaluation loop — generate candidate views, test against the downstream task, keep provenance and uncertainty, revise. "The first graph we can extract is rarely the only graph worth considering."

## Key Quotes

> "We spend much less time on the step that determines what all of them can do: turning raw data into the right graph."

The thesis in one sentence. It's a claim about where the leverage lives: not in better graph learners, but in better graph *choosers*.

> "Graph representation learning normally starts with an existing adjacency structure... These methods ask: Given this graph, how should we learn from it? Graph modeling asks the question before that: What should the nodes and edges be?"

A clean division of labor between two fields that rarely acknowledge each other. The second question is data modeling, an old craft — the essay is quietly arguing that the pre-LLM tradition of schema design never went away.

> "A more practical design is to build several governed views from the same modeled data while keeping identity, evidence, time, and access rules consistent underneath."

An explicit rejection of the one-enormous-universal-graph fantasy. Multiple task-specific views over shared identity, provenance, and governance — this is essentially the materialized-view pattern applied to graphs.

> "If a deterministic query answers the question, there is no need to start with an LLM."

Buried in the survey of five uses, and the most practical line in the piece: graphs as query engines are the baseline, and GraphRAG must beat it, not merely exist.

## Key Themes

#concept #databases #data-modelling #evaluation

## Analysis

This is a maturity-signals essay, and a good one. Its move — "the interesting problem is upstream of the fashionable one" — is the same shape as arguments that evaluation, not generation, is the real bottleneck in agentic software. And its evidence is unusually honest: it doesn't just advocate, it cites a benchmark where schema-derived graphs *hurt* and structural adaptation helped, which makes "the graph is part of the learning problem" concrete rather than rhetorical.

The weakest seam is the "graph-native AI" brand itself: the essay spends most of its length on the diagnosis and leaves the proposed system (propose, test, maintain several graph views) as a wishlist. Nobody knows how to automate that loop yet — candidate generation, downstream scoring, and provenance-preserving maintenance each look like hard research problems on their own. But the framing is right, and it gives a name to a decision that teams currently make by accident.

It also corrects a bias in this wiki's graph coverage. The graph notes here mostly cover graphs *serving agents* ([[GraphRAG]], [[Context Graphs]]) or graphs *failing to* ([[Your Knowledge Graph Is Making Your Agent Dumber]]). This essay explains why both outcomes happen: the graphs in question were extracted, not designed. The failure in the Graphify takedown — semantic fragmentation, stale high-confidence edges — is exactly what the 26-task RDL study predicts, arrived at empirically from the other direction.

## Relations

- Strengthens [[GraphRAG]] by adding the caveat the original write-up lacks: GraphRAG's entity-extraction pipeline *is* a graph-modeling choice, and recent benchmarking found cases where it underperformed vanilla RAG — so graph construction, retrieval, and generation must be evaluated together.
- Complicates [[Your Knowledge Graph Is Making Your Agent Dumber]]: Vijayan's noise tax on auto-extracted graphs is a specific instance of this essay's "information overload and semantic fragmentation" diagnosis, suggesting the fix is modeling discipline, not abandoning graphs.
- Nuances [[Context Graphs]]: typed edges capturing *why* decisions were made are a graph-modeling choice about what a relationship means — this essay supplies the "which graph should we build" question that the context-graph essay assumes answered.
- Complements [[Agent Memory]] with the design question of which parts of agent state benefit from being a graph: planning, memory, tool use, and multi-agent coordination are the four uses the graph-agent literature organizes around.

---
*Sources: [[raw/from-raw-data-to-graph-native-ai]], [[summary/from-raw-data-to-graph-native-ai]]*
*Last updated: 2026-10-03*
