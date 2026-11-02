---
url: https://tinysuite.dev/blog/how-to-add-memory-to-ai-agent/
title: "How to Add Long-Term Memory to an AI Agent (Step by Step)"
author: TinySuite
date_fetched: 2026-11-02
date_published: 2026
topics:
  - agent-memory-and-context
  - mcp-and-tool-protocols
---

A vendor tutorial (TinySuite, makers of TinyContext) that walks through adding long-term memory to an agent in three ways: as an MCP server plugged into Claude Code, Cursor or Codex; as a ~60-line Python library in your own agent; and over an HTTP API from any language. TinyContext keeps facts in one local SQLite file with a local ONNX embedding model — no vector database, no hosted account.

The core prescription is tool-shaped, not model-shaped: give the agent two tools (save short facts, recall relevant facts), then tell it when to use them — recall at the start of every task, save durable facts, update rather than duplicate, never save secrets. Tools alone aren't enough; most agents won't save anything unless instructed, so five lines in CLAUDE.md/AGENTS.md do the steering.

Memory is split into profile facts (attached to every recall) and episodic facts (searched by relevance), and the store should hold conclusions, not transcripts — one self-contained sentence per fact, updated rather than appended. Recall is hybrid BM25 keyword plus embedding search under a token budget (~800 tokens default), which the vendor's benchmark claims finds exact terms like "Postgres 16" that embedding-only search misses, at p95 under 13 ms up to 20,000 memories.

The comparison table is the interesting part: full chat history (cost grows, lost at session end), running summaries (details erode, can't correct one fact), memory files like CLAUDE.md (every line costs tokens in every session, nothing is searched), DIY vector DBs (another service plus dedup/update/budget work), and Mem0/Zep (extra LLM call per write, second database). Honest limits: TinyContext suits thousands to tens of thousands of memories, not millions; graph relationships want Zep; managed SLAs want Mem0.
