---
url: https://www.oreilly.com/radar/agent-memory/
title: Agent Memory
author: Angie Jones
date_fetched: 2026-07-05
date_published: 2026-06-29
site: O'Reilly Radar
---

# Agent Memory

Angie Jones writes about agent memory after spending time with Richmond Alake of Oracle, who has been "in the trenches working on agent memory at Oracle." Originally published on her LinkedIn, republished with permission on O'Reilly Radar. 14-minute read.

## The Problem

LLMs are "stateless by design, meaning they have no memory or awareness of past interactions." Chat interfaces fake continuity by resending the entire conversation history as one giant prompt — "behind the scenes, your agent takes the entire conversation history and resends *all of it*." This works for short chats but eventually produces a "giant blob of context" where details become muddled.

## Seven Memory Types

Jones distinguishes seven distinct categories of agent memory:

### Conversational Memory
Stores messages exchanged between user and assistant. The most common first attempt: appending prior messages to the prompt. Jones warns this "is not really memory engineering" — it works for short conversations but breaks down as context grows.

### Semantic Memory
Stores durable facts that outlive specific conversations — user preferences, system requirements, domain knowledge. Vector search enables retrieval by meaning rather than exact wording. "What time does the store close?" and "When does the shop shut?" should retrieve the same fact.

### Episodic Memory
The "what happened" layer — stores events, workflows, and sequences. Useful for debugging, auditing, and long-running workflows. Often benefits from structured database storage rather than vector search alone, because ordering matters.

### Procedural Memory
About *how* to do things — reusable approaches and processes. "That's powerful because agents are often asked to operate in messy real-world environments" where the same approach applied to different situations yields value.

### Entity Memory
Facts about specific people, accounts, projects, tickets, or objects. "If I ask, 'What do we know about Acme Corp?' I don't want every memory in the system." Requires scoped, filtered retrieval — not just semantic similarity.

### Working Memory
A short-term scratchpad for the current task. Not all temporary thoughts deserve permanent storage; otherwise, "the memory store gets noisy very quickly." The distinction between working memory and durable memory is a filtering problem.

### Summary Memory
Compresses long threads into compact representations for limited context windows. Instead of sending 80 turns of conversation, the agent sends a condensed version. Jones notes this is the type "many agent users are familiar with" because it's what most chat interfaces already do.

## Why Memory Is Hard

Jones identifies the core challenges:

- **Judgment over storage**: "The hard part is judgment, not storage" — deciding what's durable vs. transient, when to overwrite vs. timestamp, how much to retrieve
- **Retrieval balance**: Too little context and the agent is uninformed; too much and it's overwhelmed
- **Memory leaks**: Scoping memories across users, tenants, and sessions to prevent cross-contamination
- **Measurement**: How do you know whether memory actually helps? "Bad memory is worse than no memory"

## Oracle's OAMP (Oracle AI Agent Memory Package)

Built on Oracle AI Database 26ai, OAMP integrates embeddings, JSON documents, text search, and SQL in a single database. Key primitives: users/agents (ownership scoping), memories, threads, context cards, summaries, vector search, and database-backed persistence.

The most interesting design choice is **context cards** — instead of passing raw thread history to the model, `get_context_card()` produces a compact block of relevant memory. The prompt structure becomes: system instruction + memory context card + user query. This is the cleanest production answer I've seen to "how do you actually inject memory into the prompt?"

**Automatic memory extraction** can inspect conversation messages and extract durable facts automatically, configured via parameters like `memory_extraction_frequency` and `enable_context_summary`. This is the "write" side of the memory lifecycle — and the side most open-source implementations ignore.

## Teaching an Agent About a Database

Jones's most original section: using memory to teach an agent an unfamiliar database schema. The agent scans catalog tables (ALL_TABLES, ALL_TAB_COLUMNS) and converts technical metadata into natural-language memories. "You can teach it by turning your system's metadata into memory." This inverts the usual RAG pattern — instead of retrieving from human-written docs, the agent *generates* its own documentation from system metadata and stores it as memory.

## Key Quotes

> "LLMs are stateless by design, meaning they have no memory or awareness of past interactions."

The premise everything builds from. Memory isn't a model capability — it's infrastructure.

> "The most common first attempt is to keep appending prior messages to the prompt. This is not really memory engineering."

Jones draws the line between naive context-stuffing and actual memory architecture. Most "agent memory" products are still on the wrong side of it.

> "Real agent memory requires deciding what should be stored, where it should be stored, how it should be retrieved, and when it should be updated or forgotten."

The lifecycle view. Write, retrieve, update, forget — all four must be designed, not just the retrieval half.

> "The hard part is judgment, not storage."

The single most important sentence in the article. Anyone can build a vector database. Almost nobody knows what belongs in it.

> "Bad memory is worse than no memory."

The corollary. A confident wrong answer is more dangerous than admitting ignorance.

> "You can teach it by turning your system's metadata into memory."

The metadata-as-memory pattern. Private schema → natural-language memories → retrievable context. No pre-training required.
