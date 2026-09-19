---
url: https://claude-code-from-source.com/
title: "Claude Code from Source"
author: not stated
date_fetched: 2026-09-19
date_published: not stated
topics:
  - claude-code
  - agent-architecture
---

The landing page for an 18-chapter book titled "How Anthropic built the most widely used AI coding agent," distilled from the full TypeScript source of Claude Code. The provenance claim is the hook: when Claude Code shipped on npm, its `.js.map` source maps carried a `sourcesContent` field with the original TypeScript — nearly two thousand files — and the book's authors read all of it. The page sells the book by advertising six architecture areas and a "10 patterns that make it work" list.

The six advertised areas: an agent loop driven by a single async generator (streaming, tool execution, error recovery, context compression across four layers); a 14-step tool-execution pipeline with permission resolution, speculative execution, and concurrent batching classified by safety; multi-agent orchestration where sub-agents share prompt-cache prefixes for a claimed 95% cost cut, plus fork agents, coordinator mode, and swarm teams with mailbox messaging; file-based memory without a database, using an LLM-powered recall side-query (a Sonnet call) that the authors claim beats embedding search, with four memory types and staleness warnings; performance engineering (240ms startup via parallel I/O, slot reservation, bitmap pre-filters for fuzzy search); and extensibility/security — two-phase skill loading (metadata at startup, content on demand) and 27 lifecycle hooks whose configuration snapshots are frozen at startup to prevent injection.

The production story is nearly as notable as the content claims: 36 AI agents analyzed the source and wrote the book in four phases, in roughly six hours end to end, followed by an audit pass ensuring no verbatim source remained — every code block was rewritten as pseudocode with different variable names. The page carries a "purely educational" disclaimer and a parody "NO'REILLY" cover, explicitly disclaiming affiliation with O'Reilly Media. Target audiences: engineers building agentic systems (each chapter ends with five transferable "Apply This" patterns), technical leaders evaluating architectures, and the curious.
