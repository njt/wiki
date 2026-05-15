# robot.wtf

A lightweight, git-backed wiki with native MCP support and semantic search. The core idea: your AI agents and your browser read and write the same pages, the same history, the same links. It's shared memory infrastructure where humans and agents are first-class citizens on equal footing.

---

## Key Quotes

> "Your AI agents and your browser read and write the same pages, the same history, the same links."

## Key Themes

#memory #MCP #wiki #agents #knowledge-management

The human-agent symmetry is the key design insight. Most agent memory systems are either agent-only (opaque to humans) or human-only (agents can read but can't meaningfully write). robot.wtf makes both sides equal participants in a shared knowledge base. This is a different approach from [[mira-OSS]]'s first-person narrative memory or [[Context Rot]]'s five-bank architecture -- it's simpler and more transparent.

The git-backed storage means you can clone the wiki locally, inspect the full history, and merge changes. This is version control for agent memory, which most agent frameworks completely lack.

MCP integration means any MCP-compatible client (Claude, Cursor, Windsurf) can connect directly. No custom API integration needed.

The Bluesky/ATProto authentication is an interesting community choice -- it positions this as decentralized identity infrastructure, not just a wiki service.

## Critical Analysis

The simplicity is both the strength and the limitation. A flat wiki with keyword + semantic search works well for small knowledge bases, but the question is whether it scales to the kind of complex, multi-layered memory that agents need for sophisticated tasks. The five-bank architecture in [[Context Rot]] (working, history, patterns, memory_bank, books) exists because different types of memory have different lifecycles.

The volunteer/community nature of the project raises sustainability questions. It's built on An Otter Wiki, an existing open-source engine, which reduces maintenance burden but also limits customization.

For the specific use case of "shared scratchpad between human and agent" -- where both need to read and write knowledge collaboratively -- this is one of the cleanest solutions available. Compare with [[Rowboat]]'s Obsidian-compatible vault approach, which achieves similar transparency through local Markdown files.

---
*Sources: [[raw/robot-wtf]]*
*Last updated: 2026-05-14*
