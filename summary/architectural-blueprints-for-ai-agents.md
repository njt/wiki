---
url: https://gist.github.com/0c5a1403d53cba38e9f14b8f24f89230
title: "Architectural Blueprints for AI Agents"
author: unknown (senior GenAI solution architect at Nokia Networks; name garbled in the auto-transcript)
date_fetched: 2026-09-13
date_published: unknown
site: gist.github.com (ytx transcript of a conference talk)
topics:
  - agent-architecture
  - agent-memory-and-context
---

# Architectural Blueprints for AI Agents

A conference talk from Nokia Networks' private-network division (~2,000 people, building systems for mines, trains, and energy) on how a small AI team designs agents for industrial telecom systems. The driver is a labor shortage, not novelty: one network function alone carries "a configuration file that contains 27,000 configuration entries," and "simply we don't have enough engineers to manage, plan, operate, fix, support the networks." Today the team builds CLI-based, human-in-the-loop agents; autonomous background agents come later.

Two hard-won learnings anchor the talk. First, generic agent frameworks (LangChain, LlamaIndex, and most recently Atomic Agents — all "totally different abstractions") don't survive contact with industry: for agents embedded in real complex systems, you end up building your own hosting logic, orchestration logic, and architecture concept. Second, vector RAG "simply not enough": after injecting thousands of PDFs, Confluence pages, and JIRA content into various vector DBs, they concluded that semantic similarity can't deliver relationships and deep context — the fix is a deeply connected knowledge graph, i.e. Graph RAG.

His definition: an agent is "a kind of semantic execution logic" — still a class with structured input and structured output, "but in a semantic way," mixing LLM calls with native code ("the worker logic... it's unavoidable"). The blueprint has six blocks: an operation manual (identity, purpose and expectations, and an ontology — "a schema for the world" — stored externally via a repository pattern, so "an agent can build another agent"), structured I/O (the Instructor library, because "LLMs are really bad at structured outputs"), a memory ladder (naive queue → LLM summaries → vectorized memory → graph memory, "the highest complexity but the most sophisticated"), environment/context providers (four injection pipelines: DITA XML documentation, a live Kubernetes-cluster scan, functional requirements, and failures/bug fixes/support incidents — all correlated into one property graph), a toolbox (tools must be designed, not API wrappers; externalize via MCP because "we cannot build all the tools that we need"), and the worker logic itself.

Orchestration is credited to Anthropic's blog: router + agent handoff, parallel execution + a synthesizer agent, and evaluator/observer loops ("I give you agent five rounds, five iterations to try to complete the task"). The engines are Neo4j and embeddable KùzuDB; the demo visualizes an agent environment as a property graph and shows vector search to seed nodes plus graph traversal of their neighborhoods. On evaluation (Q&A): domain experts supply the meaningful questions and answers, evaluator agents automate the comparison — "it can be just an Excel file" — and produce a quality score.

The gist's summary closes with a pointed critique section: agents that "run commands" on live networks get zero safety discussion, the talk contains no metrics, the "RAG not enough" claim is asserted rather than benchmarked, graph drift and ontology maintenance go unaddressed, and model choice / data sovereignty are never mentioned despite an on-prem, customer-data-heavy business.
