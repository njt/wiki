---
url: https://github.com/HtooTayZa/sawtooth-memory
title: Sawtooth Memory
author: HtooTayZa
date_fetched: 2026-06-09
date_published: 2025
topics:
  - misc
---

# Sawtooth Memory — Deep Analysis

## Project Overview

sawtooth-memory is an async, non-blocking hierarchical memory framework for LLM agents, distributed as a Python package (v0.2.0). It addresses two problems simultaneously: (1) main-thread latency caused by synchronous LLM summarization of conversation history, and (2) the "Lost in the Middle" hallucination problem where summaries silently drop critical deterministic values (UUIDs, connection strings, file paths). The solution: a four-tier memory stack with background async compression that returns control to the main agent loop in <10ms while an asyncio worker performs LLM summarization off the critical path.

~6,800 lines of Python. MIT license. Python 3.11+. Published on PyPI.

## Architecture

### Four-Tier Memory Model

The core insight is separating what gets summarized from what gets preserved:

**L0 — SystemPrompt** (immutable): Agent persona, tool schemas, output format constraints. Never compressed, never evicted. Set once at initialization.

**L1 — WorkingMemory** (sliding window): The most recent N raw conversation messages. This is what the agent loop adds to. When the soft token limit is crossed, the oldest `chunk_size` messages are sliced off and enqueued for background compression.

**L1.5 — EntityLedger** (deterministic KV store): Before summarizing L1 messages, the compression LLM extracts exact deterministic values (UUIDs, database IDs, file paths, connection strings, numeric results) into a flat key-value dictionary. This ledger is injected into every prompt build, guaranteeing 100% recall of critical facts that would otherwise be lost in compression. Each key stores a *list* of historical values (rolling window, max 10) rather than overwriting — so if `connection_id` changes from "conn-A" to "conn-B", both are preserved with `__history` provenance.

**L2 — ArchivalMemory** (append-only narrative): Dense, chronological narrative of older conversation turns produced by the compression LLM. Token-efficient, preserves causality (why things happened, not just what). Appended to over time — never rewritten, only grows.

When `build_prompt()` is called, these tiers are assembled in priority order: L0 system → L2 archive narrative → L1.5 entity ledger → L1 working messages. This places critical system instructions and deterministic facts closest to the attention headroom.

### Async Worker Architecture

The central architectural innovation is the decoupling of ingestion from compression:

1. `ContextManager.add_message()` appends to L1 working memory and returns instantly
2. `TokenMonitor.should_trigger_compression()` evaluates two conditions: soft token limit exceeded OR max unsummarized turns reached
3. If triggered, oldest `chunk_size` messages are sliced off, a `CompressionTask` is enqueued on an `asyncio.Queue`
4. `CompressionWorker._loop()` pulls tasks from the queue and calls the compression LLM
5. Results are merged: narrative appended to L2, entities upserted into L1.5
6. Events are emitted at each stage (cycle start, L1 eviction, L2 summary generated, cycle complete/failed)
7. A journal handler writes completed cycles to append-only JSONL

The hard limit is a failsafe: if the compression worker can't keep up (slow model, API rate limits), `fallback_truncate` brutally discards the oldest messages rather than crashing the agent with context overflow.

### Debouncing

The `TokenMonitor` uses a debounce lock (`_is_compression_queued`) to prevent flooding the worker queue. Once compression is triggered, the lock is held until the worker emits a `compression.cycle_complete` or `compression.cycle_failed` event, at which point the lock is released by the event subscriber. This prevents the pathological case where every `add_message()` call triggers a new compression task while the first is still running.

## Key Techniques

### 1. Dual-extraction compression prompt

The compression system prompt (in `compressor.py`) asks the LLM to do two things simultaneously:
- Write a dense chronological narrative (traditional summarization)
- Extract all exact deterministic values into a flat KV dictionary (entity extraction)

This is more efficient than two separate LLM calls, and the structured output format (JSON schema enforced via tool-use for Anthropic, `response_format: json_object` for OpenAI, `responseMimeType: application/json` for Gemini) ensures consistent parsing.

### 2. Pre-processing noise removal

Before sending to the compression LLM, `_prune()` strips three categories of token-wasting content:
- Base64 strings >80 chars (common in tool outputs)
- Python/JS stack traces (verbose but rarely informative)
- Long hex runs (binary output dumps)

This is done with compiled regex, not an LLM call — zero latency, zero cost.

### 3. Entity conflict preservation

The `EntityLedger.upsert()` method has a subtle design: when the same key is extracted across multiple compression waves with different values, it appends to a history list rather than overwriting. When `to_json_str()` renders for prompt injection, it outputs the latest value as the primary key plus a `__history` companion key when multiple values exist. This means the agent sees `connection_id: "conn-B"` but can also access `connection_id__history: "conn-A"` if needed.

### 4. Orphaned ToolMessage sanitization (LangGraph adapter)

A clever edge-case fix in the LangGraph adapter: when an AIMessage with `tool_calls` is evicted from L1 and compressed into the L2 narrative, any corresponding `ToolMessage` children become orphaned — their `tool_call_id` references a parent that no longer exists in the prompt. Sending orphaned ToolMessages to strict cloud APIs causes HTTP 400 errors because every `tool_call_id` must match a preceding `AIMessage.tool_calls[].id`. The adapter's `get_compiled_prompt()` runs a three-pass sanitization: materialize raw dicts → collect active tool_call_ids → drop orphaned ToolMessages. Clean solution to a problem that would otherwise surface as mysterious API errors.

### 5. Provider adapter pattern

The compressor backend uses a `ProviderAdapter` Protocol (structural subtyping, not inheritance) with concrete adapters for OpenAI, Anthropic, and Gemini. Each adapter is a pure data-construction object (no I/O) that builds provider-specific headers, payloads, and parses responses into a normalized `(parsed_dict, total_tokens)` tuple. The Anthropic adapter uses tool-use (not JSON mode) to enforce structured output because Anthropic's tool-use is GA and stable. The OpenAI adapter uses `response_format: {"type": "json_object"}`. This makes the compression backend trivially swappable.

### 6. Event-driven observability without coupling

The event system uses plain `dataclasses` (not Pydantic) for speed — no validation overhead on every event. The `EventBus` is a singleton with GC-shielded background tasks (stores `asyncio.Task` references in a `set` to prevent Python 3.11+ garbage collection of fire-and-forget tasks). Handlers are subscribed by event type string, allowing the journal, monitor, and any user code to observe the system without coupling.

## Design Decisions

### Optimized for: main-thread latency

Every architectural choice flows from the goal of minimizing time spent on the critical path. `add_message()` does token counting (local tiktoken, no API call), appends to a list, checks thresholds, and optionally enqueues a task — all in microseconds. The compression LLM call happens on a background asyncio task. The benchmark shows 11.3x faster main-thread execution vs synchronous summary memory.

### Sacrificed: compression quality for speed

The compression model is expected to be a small, fast model (default: `phi4-mini:latest` for local Ollama). Summaries from a small model will be lower quality than summaries from a frontier model. The trade-off is that the entity ledger (L1.5) preserves exact values regardless of summary quality — so even if the narrative is mediocre, the agent still has the critical facts.

### Sacrificed: memory state is NOT thread-safe

Despite the DOCUMENTATION claiming `asyncio.Lock()` protection, the actual code uses no locks. `MemoryState` is a plain Pydantic model. The worker and the main thread both mutate the same state object. In an asyncio single-threaded model this mostly works because task switching only happens at `await` points, but the design implicitly assumes the worker won't be mutating state while `build_prompt()` is reading it. This is a real gap — the code relies on cooperative multitasking guarantees rather than explicit synchronization.

### Design choice: local-first, cloud-optional

The default backend is local Ollama. Cloud (OpenAI/Anthropic/Gemini) is an optional config. This makes the library usable in air-gapped environments without API keys, but also means the default compression quality depends on having a capable local model.

### Design choice: JSONL journal over database

Compression cycles are persisted to a JSONL file rather than SQLite or a proper database. This makes the journal human-readable and trivially parseable (one JSON object per line), but limits queryability and concurrent access. The DOCUMENTATION acknowledges this as a Phase 3 item (Redis/Postgres adapter).

## Comparison Notes

**vs. LangChain ConversationSummaryMemory**: LangChain's approach is synchronous — the app blocks while the LLM summarizes. Sawtooth moves this to a background worker. LangChain produces a single summary string; Sawtooth separates the summary (L2) from exact values (L1.5), preventing hallucinated UUIDs.

**vs. Three Tier Memory (the 108K-line C# project)**: Similar hierarchical concept but different tiers. Three Tier uses hot-memory constitution + domain experts + cold knowledge base. Sawtooth uses system prompt + working memory + entity ledger + archival narrative. The key difference: Sawtooth extracts *deterministic entities* into a separate tier (L1.5) while Three Tier relies on domain-expert agents for retrieval. Sawtooth's entity ledger is a simpler, more mechanical guarantee.

**vs. Mnemo**: Mnemo builds a knowledge graph from conversations (entities + relationships). Sawtooth extracts a flat KV dictionary. Mnemo's graph is richer (relationships between entities) but requires graph traversal. Sawtooth's flat dict is simpler (just inject the values) but misses relational structure. Complementary approaches — Mnemo for semantic knowledge, Sawtooth for operational context compression.

**vs. Memory Mechanism (xAI taxonomy)**: Sawtooth's L1 maps to "session memory," L1.5 to a hybrid of "semantic memory" (deterministic facts), and L2 to "episodic memory" (compressed narrative). The taxonomy distinction between "instruction memory" and "learning memory" isn't directly modeled — Sawtooth treats everything in the same compression pipeline.

**vs. napkin / Claude-Mem**: Those are file-based approaches (a markdown file or a summary file as memory). Sawtooth is a middleware that sits between the agent loop and the LLM API call, managing memory automatically rather than requiring the agent to explicitly write/read from a file.

## File Structure

```
sawtooth_memory/
├── __init__.py          (47 lines)  Public API exports
├── config.py            (106 lines) Pydantic config models
├── middleware.py        (534 lines) ContextManager — main public API
├── compressor.py        (377 lines) OllamaCompressor + CloudCompressor
├── worker.py            (321 lines) Background asyncio compression worker
├── state.py             (207 lines) 4-tier Pydantic state model
├── monitor.py           (230 lines) tiktoken-based token monitoring
├── journal.py           (221 lines) Async JSONL journal
├── exceptions.py                    Custom exception hierarchy
├── providers/
│   ├── adapters.py      (365 lines) OpenAI, Anthropic, Gemini adapters
│   └── factory.py       (33 lines)  Adapter factory
├── events/
│   ├── types.py         (99 lines)  Dataclass event types
│   ├── bus.py           (101 lines) Async event bus with GC shielding
│   └── handlers.py      (35 lines)  Event handlers
└── integrations/
    └── langgraph/
        ├── adapter.py   (262 lines) Bidirectional LangChain message bridge
        └── graph.py     (219 lines) Pre-built LangGraph StateGraph
tests/                              pytest-asyncio test suite
examples/                           Basic usage + LangGraph examples
scripts/benchmark_memory.py         Comparative benchmark harness
```

## Raw Notes

- The name "sawtooth" likely references the sawtooth wave pattern — periodic sharp drops (compression evictions) followed by gradual rises (message accumulation), mirroring the token count graph over time.
- The project explicitly calls out Phase 3 roadmap items: multi-agent memory pooling, semantic vector L3, and Redis/Postgres adapters.
- The `max_unsummarized_turns` config allows turn-count-based batching instead of (or in addition to) token-count-based compression triggers — useful when small messages accumulate many turns without hitting the token limit.
- The `explain_prompt()` method provides a deterministic audit trail of what's in the current prompt and why — this is unusual for a memory library and shows a focus on debuggability.
- The LangGraph adapter does message-ID deduplication so it's safe to call `sync_state()` on every graph iteration (including cycles) without double-ingesting.
- The benchmark claims 11.3x faster main-thread execution but the baseline is local Ollama at 64 seconds for a 20-message conversation — unusually slow for a sequential summary pattern and likely reflects a very small local model.
