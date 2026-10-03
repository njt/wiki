---
url: https://www.oreilly.com/radar/from-raw-data-to-graph-native-ai/
date_fetched: 2026-10-03
---

I’ve been thinking about a gap in the way we discuss graphs in AI. Most of the conversation starts after the graph already exists. We talk about graph neural networks, GraphRAG, graph agents, and graph foundation models. We spend much less time on the step that determines what all of them can do: turning raw data into the right graph.

Suppose an organization has customer records in tables, support conversations in documents, product telemetry in event streams, and incident histories in tickets. Before any of this can be used as a graph, someone has to decide what counts as an entity, what a relationship means, how time is represented, and which source to trust when records disagree. These choices shape every prediction, retrieval result, and agent decision that follows.

This is what I mean by graph-native AI. It’s an approach that treats graph modeling as a first-class stage between raw data and the different systems that may use it. The graph isn’t simply an input to one model. It becomes a shared representation that can support queries, prediction, retrieval, agents, and, potentially, foundation models.1

## A graph is a model of the data

At its simplest, a graph consists of nodes and edges, which may also carry features. Real systems also need types, timestamps, provenance, confidence, permissions, and other context. Describing the finished graph tells us very little about how we should get there.

Take customer-support data as a simple example. Should a company be represented as one node or as a set of legal entities that changes over time? Should an email be an edge between two people, a document node connected to its authors, or evidence for claims extracted from the text? Is a purchase an ongoing relationship or an event with a timestamp? If two records refer to the same customer, should we merge them, link them as possible matches, or keep them separate?

There’s no single answer. An entity graph may be useful for identity and relationships. An event graph may be better for process and temporal analysis. An evidence graph may be better for retrieval and provenance. The right choice depends on what we want the system to do.

## The step before graph learning

Graph representation learning normally starts with an existing adjacency structure. Embeddings learn vectors for nodes. Graph neural networks pass information across neighborhoods. Autoencoders reconstruct structure or attributes. Graph transformers combine local graph structure with longer-range attention. These methods ask: Given this graph, how should we learn from it?2 Graph modeling asks the question before that: What should the nodes and edges be?

Relational deep learning makes this distinction concrete. It maps rows in relational tables to nodes and primary-foreign-key links to edges, producing a temporal heterogeneous graph that a GNN can learn from. This can remove a good deal of manual joining and feature engineering. It also makes an assumption that deserves attention: A database schema designed for storage and transactions is also a useful graph for machine learning.3

Recent work tests that assumption across 26 relational tasks. Graphs derived directly from database schemas sometimes suffered from information overload and semantic fragmentation. The authors improved performance by adapting the structure: removing distracting connections and adding dependencies that the original schema did not capture. I find this result important because it makes the graph itself part of the learning problem. Adding more edges is not always helpful. What matters is whether the structure supports the reasoning required by the task.4

Graph construction therefore needs its own evaluation loop. We can generate a few plausible graph views, test them against the downstream task, inspect failures, and revise the structure. We should also keep provenance and uncertainty so that we know where an edge came from and how confident we are in it. The first graph we can extract is rarely the only graph worth considering.

## What the graph can support

Once the graph has been modeled and checked, several paths open. I don’t think this means every application should use one enormous universal graph. A more practical design is to build several governed views from the same modeled data while keeping identity, evidence, time, and access rules consistent underneath. Here are five ways to use graphs in an AI application.

- **Query and analytics:**A graph database and ordinary graph algorithms may already be enough. Traversals, path queries, neighborhood aggregation, centrality, and community detection can reveal relationships that are awkward to see once the data is flattened into rows. This is also a useful baseline: If a deterministic query answers the question, there is no need to start with an LLM.
- **Prediction and graph representation learning:**Embeddings, GNNs, graph autoencoders, and graph transformers can make predictions at the node, edge, subgraph, or whole-graph level. Common examples include fraud detection, recommendation, molecular-property prediction, and link prediction. Here, the graph gives the model an explicit view of which entities are related and how information should move between them.
- **Retrieval and GraphRAG:**In this case, the graph is used as an index rather than as training data. Microsoft’s GraphRAG work builds an entity graph and community summaries to answer broad questions over a corpus that ordinary chunk retrieval may handle poorly.- 5Other approaches retrieve a task-specific subgraph and pass it to an LLM. This can be useful, but the extra graph pipeline has to improve the final result. Recent benchmarking found cases where GraphRAG underperformed vanilla RAG and argued that graph construction, retrieval, and generation should be evaluated together.- 6
- **Graph-augmented agents:**Agents have memory, plans, tools, state transitions, and sometimes relationships with other agents. Some of this state is naturally relational. A graph can make it persistent, inspectable, and easier to update across turns. The emerging literature organizes these uses around planning, memory, tool use, and multi-agent coordination. The practical design question is which parts of an agent’s state benefit from being represented as a graph.- 7
- **Graph foundation models:**The goal here is to pretrain a model that can transfer across graph tasks, datasets, or domains. Current work combines graph backbones, self-supervised objectives, adaptation mechanisms, and sometimes language models. The difficult part is that graph semantics vary widely. Nodes and edges can represent very different things across molecular, transaction, and knowledge graphs. Transfer across these domains requires methods that bridge differences in structure and semantics, supported by suitable training data.- 8

## What better graph modeling would look like

When I say that we need to crack graph modeling, I don’t mean a universal converter that produces one correct graph. The more useful goal is a system that can propose, test, and maintain several graph views of the same raw data.

Such a system would need to do a few things well. It should identify possible entities and relationships in tables, text, events, images, and existing schemas. It should retain type, time, provenance, confidence, and permissions. It should be able to suggest alternatives instead of quietly committing to one structure. It should compare those alternatives using downstream quality, cost, stability, and human constraints. And it should keep the graph current as the source data and the organization’s understanding of it change.

The downstream applications provide useful feedback. Prediction errors can point to missing relationships. Retrieval failures can expose poor granularity or disconnected evidence. Agent failures can show that important state transitions are absent. Weak transfer across datasets can reveal schemas that do not align. In this view, the graph is evaluated by what it helps the system do.

Today, a team will often build a graph for one application: a fraud graph, a recommendation graph, or a knowledge graph for a chatbot. I think there is a larger opportunity. If the underlying graph modeling is done carefully, the same raw data can support several graph views and several applications without rebuilding identity, provenance, and governance each time.

We already have strong methods for learning from graphs. The harder and less settled problem is deciding which graph we should build. If we make that step systematic and measurable, graph modeling can become a field in its own right and a shared foundation for analytics, prediction, retrieval, agents, and foundation models.

### Footnotes

- Arijit Khan, Longxu Sun, Xin Huang, “LLMs+Graphs: Toward Graph-Native, Synergistic AI Systems,” arXiv, June 2026. ↩︎
- William L. Hamilton, *Graph Representation Learning*(Morgan & Claypool, 2020). ↩︎
- Matthias Fey et al., “Relational Deep Learning: Graph Representation Learning on Relational Databases,” arXiv, submitted Dec 2023. ↩︎
- Yao Cheng and Siqiang Luo, “What Makes a Desired Graph for Relational Deep Learning?” arXiv, submitted June 2026. ↩︎
- Darren Edge et al., “From Local to Global: A Graph RAG Approach to Query-Focused Summarization,” arXiv, revised Feb. 19, 2025. ↩︎
- Zhishang Xiang et al., “When to Use Graphs in RAG,” arXiv, revised Feb 2026. ↩︎
- Yixin Liu et al., “Graph-Augmented Large Language Model Agents,” arXiv, revised Aug 2025. ↩︎
- Zehong Wang et al., “Graph Foundation Models: A Comprehensive Survey,” arXiv, May 2025.
 ↩︎
