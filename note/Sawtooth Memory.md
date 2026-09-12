# Sawtooth Memory

An async, non-blocking hierarchical memory framework for LLM agents that solves two problems at once: main-thread latency from synchronous summarization, and the "Lost in the Middle" hallucination problem where summaries silently drop UUIDs, connection strings, and other deterministic values. The core insight: decouple message ingestion from compression by running summarization on a background asyncio worker, and extract critical exact values into a separate immutable ledger (L1.5) before compression, guaranteeing 100% fact recall regardless of summary quality. Drop-in middleware between your agent loop and LLM API call — `add_message()` returns in microseconds, `build_prompt()` assembles a four-tier optimized context. Supports local Ollama (default) or cloud backends (OpenAI, Anthropic, Gemini). ~6,800 lines of Python, MIT license, on PyPI.

## Architecture

The memory model has four tiers, assembled in priority order on `build_prompt()`:

- **L0 — SystemPrompt**: Immutable persona + tool schemas + output constraints. Never compressed, never evicted.
- **L2 — ArchivalMemory**: Append-only dense narrative of older conversation turns, produced by the background compression worker. Preserves causality (why things happened).
- **L1.5 — EntityLedger**: Flat key-value dictionary of exact deterministic values extracted during compression — UUIDs, database IDs, file paths, connection strings, numeric results. Each key stores a *history list* (rolling window, max 10) so value changes are preserved with `__history` provenance. Injected directly into the system message.
- **L1 — WorkingMemory**: Sliding window of the most recent raw conversation messages.

The non-blocking execution model:

```
Main Thread                          Background Worker
───────────                          ─────────────────
add_message() → append L1, return    (instant, ~microseconds)
                                     TokenMonitor checks soft/hard limits
                                     Slice oldest chunk_size messages
                                     Enqueue CompressionTask
                                     → LLM compress (Ollama/Cloud)
                                     → Merge narrative → L2
                                     → Upsert entities → L1.5
                                     → Emit cycle_complete event
                                     → Journal write (JSONL)
build_prompt() → stitch L0+L2+L1.5+L1
```

Key source files: `middleware.py:229-268` (add_message flow), `worker.py:143-251` (compression loop + state merge), `state.py:65-186` (EntityLedger with conflict preservation), `compressor.py:84-162` (OllamaCompressor with noise pruning).

## Key Techniques

**Dual-extraction compression prompt** (`compressor.py:27-52`): The system prompt asks the compression LLM to simultaneously produce a chronological narrative AND extract exact deterministic values into a flat KV dict — two tasks in one LLM call. Structured output enforced differently per provider: Anthropic tool-use (`store_compression_result` tool with `tool_choice: {type: "tool"}`), OpenAI `response_format: json_object`, Gemini `responseMimeType: application/json`.

**Pre-processing noise removal** (`compressor.py:58-76`): Three compiled regexes strip base64 blobs >80 chars, Python/JS stack traces, and long hex runs from messages before they hit the compression LLM — saving tokens and improving summary quality with zero latency cost.

**Entity conflict preservation** (`state.py:122-154`): `EntityLedger.upsert()` appends new values for existing keys to a history list rather than overwriting. Duplicate consecutive values are skipped to avoid duplication across overlapping compression waves. `to_json_str()` renders the latest value as primary + `__history` companion key when multiple values exist.

**Orphaned ToolMessage sanitization** (`integrations/langgraph/adapter.py:188-261`): When an AIMessage with `tool_calls` is evicted from L1 into L2, its child ToolMessages become orphaned (their `tool_call_id` references a parent no longer in the prompt). Sending these to strict APIs causes HTTP 400. The adapter's `get_compiled_prompt()` runs a three-pass sanitization to detect and drop orphaned ToolMessages.

**Debounce lock** (`monitor.py:72-77`): Prevents queue flooding — once compression is triggered, `_is_compression_queued` blocks further triggers until the worker emits `compression.cycle_complete` or `compression.cycle_failed`, resetting the lock via event subscription.

## Design Decisions

**Optimized for main-thread latency, sacrificed compression quality**: Default compression model is `phi4-mini:latest` (small local model). Summaries will be lower quality than frontier models produce, but the entity ledger (L1.5) preserves exact values regardless. Benchmark claims 11.3x faster vs synchronous summary memory for local models.

**Local-first, cloud-optional**: Default backend is Ollama (`localhost:11434`). Cloud is an opt-in config. Makes the library usable in air-gapped environments but means compression quality depends on a capable local model being available.

**No locks despite concurrency**: DOCUMENTATION claims `asyncio.Lock()` protection, but the actual code uses no synchronization. `MemoryState` is a plain Pydantic model mutated by both the main thread and the background worker. Works because of asyncio's cooperative multitasking (task switches only at `await`), but is a real gap — `build_prompt()` reading state mid-way through a `_merge()` could see inconsistent data.

**JSONL journal over database**: Compression cycles are persisted to append-only JSONL (one JSON object per line) rather than SQLite. Human-readable and trivially parseable, but limits queryability and concurrent access. Phase 3 roadmap promises Redis/Postgres adapters.

**Provider adapter Protocol over ABC**: `ProviderAdapter` is a `typing.Protocol` with `@runtime_checkable` — structural subtyping, not inheritance. Each adapter is a pure data-construction object (no I/O). Makes the compression backend trivially swappable.

## Comparison Notes

- **vs. LangChain ConversationSummaryMemory**: LangChain blocks the main thread during LLM summarization; Sawtooth offloads it. LangChain produces a single summary string; Sawtooth separates narrative (L2) from exact values (L1.5), preventing hallucinated facts.
- **vs. Three Tier Memory**: Similar hierarchical concept but different tier semantics. Three Tier uses domain-expert agents for retrieval; Sawtooth uses a deterministic entity ledger. Sawtooth's L1.5 is a simpler, more mechanical guarantee — no retrieval step, just direct injection.
- **vs. Mnemo**: Mnemo builds a knowledge graph (entities + relationships); Sawtooth extracts a flat KV dict. Mnemo is richer semantically; Sawtooth is simpler operationally. Complementary — Mnemo for long-term semantic memory, Sawtooth for real-time context compression.
- **vs. napkin / Claude-Mem**: Those are file-based — the agent explicitly reads/writes a memory file. Sawtooth is middleware that manages memory automatically between the agent loop and the LLM API.
- **vs. Zero-Mem**: [[Zero-Mem]] eliminates LLM calls from memory entirely — no compression, no summarization, no extraction. Instead of compressing old turns into narratives (L2) and extracting values into a ledger (L1.5), Zero-Mem keeps raw turns as the only artifact and runs deterministic retrieval over them. The trade is Sawtooth's richer, compressed context vs. Zero-Mem's zero-token, zero-hallucination guarantee.

## Tags

#tool #project #agents #memory #context #llm #middleware #python

## Related

- [[Agent Memory and Context]] — Synthesis page: context management is the real engineering challenge
- [[Three Tier Memory]] — Similar hierarchical memory architecture (constitution + experts + cold storage)
- [[Memory Mechanism]] — xAI's five-type memory taxonomy
- [[How AI Agent Memory Works]] — Introductory overview of agent memory architecture
- [[Memory Is a Mistake]] — Critique arguing retrieval policy, not storage, is the hard problem
- [[Mnemo]] — Local-first knowledge graph sidecar, complementary approach
- [[Context Rot]] — Retrieval quality degradation over time
- [[Odysseus]] — Dual memory: working + compacted (similar compression concept)
- [[napkin]] — File-as-memory, simpler alternative
- [[Claude-Mem]] — Claude memory extraction patterns
- [[All Your Agents Are Going Async]] — Async agent trend
- [[Elements of Agentic Systems Design]] — Context and Memory as two of ten elements
- [[Local Deep Research]] — Same local-Ollama default, opposite memory model: Sawtooth is hierarchical *episodic* memory over conversation turns; LDR has no agent-memory hierarchy at all, and its long-term memory is *semantic* — an encrypted library/RAG store where downloaded sources are chunked and embedded for future searches. Useful contrast on what "memory" means for research agents vs. chat agents

---

*Source: https://github.com/HtooTayZa/sawtooth-memory | Fetched 2026-06-09 | Deep analysis from full repo clone*
