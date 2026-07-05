---
title: "How Clawdbot Remembers Everything"
url: https://x.com/manthanguptaa/status/2015780646770323543
author: Manthan Gupta (@manthanguptaa)
date_fetched: 2026-05-18
date_published: 2026-01-26
section: "Memory & Context"
---

# How Clawdbot Remembers Everything — Manthan Gupta

Full blog post at manthanguptaa.in, published 2026-01-26. 1.9M views, 135K likes, 6,294 reposts, 4.1K bookmarks.

Clawdbot (also called OpenClaw/Moltbot) is an open-source personal AI assistant (MIT licensed) created by Peter Steinberger with 32,600+ GitHub stars. It runs locally and integrates with Discord, WhatsApp, Telegram, and more. The piece is a detailed architectural walkthrough of its memory system.

## How Context Is Built

The model sees on each request:
1. System Prompt (static + conditional instructions)
2. Project Context (bootstrap files: AGENTS.md, SOUL.md, etc.)
3. Conversation History (messages, tool calls, compaction summaries)
4. Current Message

Project Context includes user-editable Markdown files injected into every request, living in the agent's workspace alongside memory files.

## Context vs Memory

**Context** = System Prompt + Conversation History + Tool Results + Attachments. Ephemeral, bounded by context window, expensive (every token counts toward API costs).

**Memory** = MEMORY.md + memory/*.md + Session Transcripts. Persistent, unbounded, cheap (no API cost to store), searchable (indexed for semantic retrieval).

## The Memory Tools

Two specialized tools:

**memory_search**: Semantic search across all memory files. Returns ranked results with path, line range, score, and snippet. Uses OpenAI text-embedding-3-small by default.

**memory_get**: Read specific lines from a memory file after searching.

There is no dedicated `memory_write` tool — the agent writes to memory using standard write and edit tools. Since memory is plain Markdown, users can manually edit files too (they auto-reindex).

## Two-Layer Memory Storage

```
~/clawd/
├── MEMORY.md          — Layer 2: Long-term curated knowledge
└── memory/
    ├── 2026-01-26.md  — Layer 1: Today's notes
    ├── 2026-01-25.md  — Yesterday's notes
    └── ...
```

**Layer 1 (Daily Logs)**: Append-only daily notes the agent writes throughout the day.

**Layer 2 (MEMORY.md)**: Curated, persistent knowledge — user preferences, important decisions, key contacts.

## How the Agent Knows to Read Memory

AGENTS.md instructions:
1. Read SOUL.md
2. Read USER.md
3. Read memory/YYYY-MM-DD.md (today and yesterday)
4. If in MAIN SESSION, also read MEMORY.md

## Indexing Pipeline

1. File saved → Chokidar detects change (1.5s debounce)
2. Chunked into ~400 token chunks with 80 token overlap
3. Each chunk embedded via OpenAI/Gemini/Local → 1536-dimension vector
4. Stored in `~/.clawdbot/memory/<agentId>.sqlite`:
   - `chunks` table (id, path, start_line, end_line, text, hash)
   - `chunks_vec` table (id, embedding) via sqlite-vec extension
   - `chunks_fts` table (text) via FTS5 full-text search
   - `embedding_cache` table (hash, vector) to avoid re-embedding

sqlite-vec enables vector similarity search directly in SQLite — no external vector database.

## Hybrid Search

Two strategies run in parallel:
- **Vector search** (semantic): finds content that means the same thing
- **BM25 search** (keyword): finds content with exact tokens (via FTS5)

Combined: `finalScore = (0.7 * vectorScore) + (0.3 * textScore)`

Results below minScore threshold (default 0.35) are filtered out. All values configurable.

## Multi-Agent Memory

Each agent gets complete memory isolation:
- `~/clawd/` — "main" agent workspace files
- `~/clawd-work/` — "work" agent workspace files
- `~/.clawdbot/memory/main.sqlite` — main agent index
- `~/.clawdbot/memory/work.sqlite` — work agent index

Markdown files (source of truth) live in each workspace. SQLite indexes (derived data) live in state directory. No cross-agent memory search by default, though workspaces are soft sandboxes.

## Compaction

When context approaches the model's limit, older conversation is summarized into a compact entry while recent messages stay intact. The summary persists to the session's JSONL transcript file, so future sessions start with compacted history.

**Automatic**: Triggers when approaching context limit. The original request retries with compacted context.

**Manual**: `/compact` command — "Focus on decisions and open questions."

## Pre-Compaction Memory Flush

LLM compaction is lossy. Before compaction triggers, a silent flush turn fires:
- System tells agent: "Store durable memories now (use memory/YYYY-MM-DD.md). If nothing to store, reply with NO_REPLY."
- Agent reviews conversation, writes key decisions/facts to memory
- Agent replies NO_REPLY (user sees nothing)
- Compaction proceeds safely

Configurable via clawdbot.yaml: reserveTokensFloor, softThresholdTokens, custom system prompt.

## Pruning

Tool results can be huge (50,000+ chars of logs). Pruning trims old outputs without rewriting history:
- **Soft trim**: Keep head + tail chars, truncate middle
- **Hard clear**: Replace old results with placeholder text
- JSONL on disk is unchanged (full outputs preserved)

## Cache-TTL Pruning

Anthropic caches prompt prefixes for ~5 minutes. When cache expires, the next request pays full "cache write" pricing. Cache-TTL pruning detects expired cache and trims old tool results before the next request, reducing re-cache cost.

## Session Lifecycle

Sessions reset based on configurable rules (default: daily). On `/new`, the session memory hook auto-saves context: extracts last 15 messages, generates descriptive slug via LLM, saves to `~/clawd/memory/YYYY-MM-DD-<slug>.md`.

## Design Principles

1. **Transparency over black boxes**: Memory is plain Markdown — readable, editable, version-controllable
2. **Search over injection**: Agent searches for what's relevant rather than stuffing everything into context
3. **Persistence over session**: Important information survives in files, not just conversation history
4. **Hybrid over pure**: Vector + keyword search together

## Related Essay

Gupta also wrote a companion essay "Memory Is a Mistake" at manthanguptaa.in/posts/memory_is_a_mistake/, arguing most AI products should not ship memory. Six concrete failure modes, retrieval policy as the hard problem, legible state over implicit memory.
