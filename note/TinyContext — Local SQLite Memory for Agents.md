# TinyContext — Local SQLite Memory for Agents

A step-by-step tutorial from TinySuite on wiring long-term memory into an agent via TinyContext, their open-source memory server: short facts in one local SQLite file, hybrid BM25 + embedding recall under a token budget, no vector database and no third-party memory hosting. It covers three integration paths — MCP for Claude Code/Cursor/Codex, a ~60-line Python library, and an HTTP API — and, unusually for a vendor post, includes a candid comparison table of the alternatives and its own scale limits.

---

The tutorial's central claim is that agent memory is a *tooling and instruction* problem, not a model problem. You install two tools (save, recall) plus five lines of steering — "recall at the start of a task, save durable facts, update rather than duplicate, never save secrets." The authors are blunt about what happens without the steering:

> Tools alone aren't enough. Most agents won't save anything unless they are told what is worth keeping.

That matches what other memory writeups in this wiki keep finding: the recall loop has to be forced, either in instructions or, more reliably, in harness code. TinyContext's Python path does exactly that — recall automatically before each turn (so the agent can't forget to look), save through tools (so the model decides what is durable). That's the same division of labour [[Three Tier Memory]] and [[Memory Is a Mistake]] argue for: deterministic retrieval, model-judged salience.

The memory taxonomy is two-bucket: **profile** facts ("prefers pnpm") attached to every recall, and **episodic** facts ("rejected Redis: too much ops") searched by relevance. The write discipline is where most memory schemes fall over, and this one has the right instincts:

> Store conclusions, not transcripts: one sentence per fact, written so it still makes sense months later. And update, don't append: when a fact changes, replace the old memory.

Supersede-with-history on update plus automatic near-duplicate skipping is the mechanism that stops a memory store from rotting into contradictions — the failure mode [[Sawtooth Memory]] and [[Memory Mechanism]] both document.

The comparison table is the honest core of the piece. Against full chat history (cost grows every turn), running summaries (details erode per rewrite, one fact can't be fixed), memory files like CLAUDE.md (every line costs tokens every session, nothing searched), DIY vector DBs, and Mem0/Zep (extra LLM call per write), TinyContext offers bounded-token hybrid recall: BM25 catches exact strings like "Postgres 16" that embeddings miss, at a claimed p95 under 13 ms to 20,000 memories. And the limits are stated rather than buried: no ANN index, so it's built for thousands to tens of thousands of memories, not millions; graphs want Zep; managed SLAs want Mem0.

Two things worth flagging as complications. First, the piece quietly strengthens the case made by [[Zero-Mem — Zero-Token Memory Operations]] and [[Zero-Mem]] that the memory stack is collapsing toward boring local infrastructure — SQLite, keyword-plus-dense retrieval, deterministic updates — rather than toward LLM-mediated memory extraction or graph stores. Second, the trust framing ("hidden memory that can't be inspected is a trust problem" — ship list and delete tools) is a governance point most memory tutorials skip, and it connects directly to the user-transparency arguments in [[Agentic Context Management]] and the privacy-driven local-first stance of [[A Home for Personal Context (Duncan Davidson)]].

One caution: the benchmark numbers (96.7% token savings, 11.4 ms median) are vendor-published and unverified. The architectural pattern, though, is sound and reproducible — and the same-store-across-Claude-Code/Cursor/Codex trick via a shared `~` SQLite path is a neat practical detail worth stealing even if you build the memory layer yourself.

**Key themes:** #concept (profile vs episodic memory; update-don't-append) #tool (TinyContext) #pattern (recall automatically, save through tools) #comparison (memory approaches trade-off table)

---
*Sources: [[raw/how-to-add-memory-to-ai-agent]], [[summary/how-to-add-memory-to-ai-agent]]*
*Last updated: 2026-11-02*
