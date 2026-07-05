---
title: "Claude-Mem"
url: https://github.com/thedotmack/claude-mem
date_fetched: 2026-05-14
section: "AI-enhanced Coding"
---

# Claude-Mem: Persistent Memory System for AI Agents

Claude Code plugin that automatically captures everything Claude does during coding sessions, compresses it with AI (using Claude's agent-sdk), and injects relevant context back into future sessions.

## Key Features

- **Persistent Context**: Memory survives across sessions
- **Progressive Disclosure**: Layered retrieval with visible token cost
- **Skill-Based Search**: Natural language queries via mem-search skill
- **Web Viewer UI**: Real-time memory stream at localhost:37777
- **Privacy Controls**: `<private>` tags exclude sensitive content
- **Citation System**: Reference past observations with IDs

## Architecture

- 5 Lifecycle Hooks: SessionStart, UserPromptSubmit, PostToolUse, Stop, SessionEnd
- Worker Service on port 37777 (Bun runtime)
- SQLite for sessions/observations
- Chroma Vector Database for hybrid semantic + keyword search
- MCP Search Tools: 3-layer workflow (search -> timeline -> get_observations)

## 3-Layer Search (10x token reduction)
- Layer 1: Compact index (~50-100 tokens)
- Layer 2: Chronological context
- Layer 3: Full observation details (500-1,000 tokens) only for filtered results

Requirements: Node.js 18+, Claude Code with plugin support, Bun runtime, SQLite 3.
Apache 2.0 license.
