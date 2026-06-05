---
url: https://github.com/pewdiepie-archdaemon/odysseus
title: Odysseus — Self-Hosted AI Workspace
author: pewdiepie-archdaemon
date_fetched: 2026-06-05
date_published: 2025
tags: [ai, agent, self-hosted, chat, local-first, fastapi, python, mcp, tool-use]
---

# Odysseus — Full Repository Analysis

## Overview

Odysseus is a self-hosted AI workspace that aims to be the open-source equivalent of ChatGPT/Claude's UI experience, running on your own hardware with your own data. It's a monolithic Python FastAPI application (~77K lines of Python across src/, routes/, services/, core/) with a vanilla JavaScript SPA frontend (~96K lines of JS in static/js/).

The project is opinionated, pragmatic, and unusually comprehensive for a personal project — it includes chat, agent tool use, model serving (cookbook), deep research, document editing, email with AI triage, calendar with CalDAV sync, notes/tasks, memory/skills, and a companion system for phone access.

## Architecture Deep Dive

### Server Architecture (Python/FastAPI)

**Entry point**: `app.py` (1100 lines) — FastAPI app creation, middleware setup (CORS, security headers, auth), lifespan management (startup/shutdown for services), static file serving, and route registration.

**Core layer** (`core/`):
- `database.py` (1979 lines) — SQLAlchemy ORM models and session management. Uses SQLite by default. Models: Session, ChatMessage, ApiToken, McpServer, User, Preset, Document, Calendar*, Email*, etc.
- `auth.py` — Auth manager with bcrypt password hashing, 2FA (TOTP via pyotp), session tokens
- `middleware.py` — Security headers, auth middleware that validates cookies and Bearer tokens
- `models.py` — Pydantic models shared across the app
- `session_manager.py` — Session CRUD operations
- `atomic_io.py` — Atomic file writes (write to temp, rename) for crash safety
- `platform_compat.py` — Cross-platform helpers (Windows vs Unix)

**Routes layer** (`routes/`, 50 files):
Each feature area has its own route file: chat_routes, session_routes, document_routes, email_routes, calendar_routes, memory_routes, skills_routes, research_routes, search_routes, mcp_routes, model_routes, task_routes, cookbook_routes, etc. Standard FastAPI routers mounted under `/api/`.

**Source layer** (`src/`, ~89 files):
The business logic. Key modules:

- `llm_core.py` (1671 lines) — The universal LLM client. Handles OpenAI-compatible API, Anthropic native API, Ollama native API, and OpenRouter. Features: SHA-256 response cache (128-entry LRU), dead-host cooldown circuit breaker (2 consecutive failures → 20s cooldown), connection pooling via shared httpx.AsyncClient (100 max connections, 30 keepalive), thread-safe health tracking.

- `agent_loop.py` (2493 lines) — The streaming agent loop. Wraps `stream_llm()` with multi-round tool execution. Two modes: (1) fenced-code-block mode for any model — the LLM writes ```toolname blocks that get parsed by regex and dispatched, (2) native function-calling mode for OpenAI-compatible APIs — uses `tool_choice: "auto"`. The ~600-line agent preamble/rules section is a masterwork of prompt engineering covering tool usage, email UID handling, cookbook procedures, UI conventions, and anti-pattern documentation.

- `agent_tools.py` (facade) → delegates to `tool_parsing.py`, `tool_schemas.py`, `tool_execution.py` (1522 lines), `tool_implementations.py` (4457 lines). The tool implementations file is the largest in the codebase.

- `chat_handler.py` — Orchestrates chat: preset resolution, context assembly (history + memory + RAG + web search + YouTube transcripts), model dispatch.

- `chat_processor.py` — Pre/post processing: hybrid memory retrieval (BM25 keyword scoring + optional ChromaDB vector similarity), RAG from personal documents, web search injection, YouTube transcript extraction.

- `memory.py` — JSON-file-based MemoryManager with Jaccard similarity for dedup, keyword extraction from chat history.

- `memory_provider.py` — Abstract MemoryProvider interface (ABC) with MemoryRecord and MemorySearchHit dataclasses. Design for pluggable backends.

- `memory_vector.py` — ChromaDB + fastembed (ONNX) vector store for semantic memory search.

- `context_budget.py` — Adaptive input-token budget: scales from 6K default to 85% of context window, capped at 200K.

- `context_compactor.py` — Self-summarization when approaching context limits (85% threshold). Uses a structured "Cursor-style" summary format: User Goal, What Was Done, Current State, Pending/Next Steps, Key Context.

- `deep_research.py` (917 lines) — IterResearch-style deep research engine. Think→Search→Extract→Synthesize loop. The LLM drives every decision: what to search, what's relevant, what's missing, when to stop.

- `mcp_manager.py` — Manages MCP server connections. Each MCP server exposes tools available to the agent. Handles sanitization of third-party MCP tool schemas for prompt injection (caps: 12 params, 40 char tokens, 300 char total hint).

- `tool_security.py` — Blocks dangerous tools, workspace confinement for file operations.

- `builtin_actions.py` (2235 lines) — Registry of automation actions (tidy sessions, tidy documents, consolidate memory, email triage) that run without LLM calls.

- `task_scheduler.py` (2286 lines) — Cron-style task scheduler for recurring agent jobs.

- `config.py` — Pydantic Settings-based configuration with env var prefix support (DATA_, LLM_, SEARCH_, SECURITY_).

- `settings.py` — Runtime settings stored in data/settings.json with a 2-second TTL cache for hot-path performance.

**Services layer** (`services/`):
Modular service implementations for memory, search, research, shell, TTS, STT, YouTube, docs, and hwfit (hardware fitness for model selection).

**MCP servers** (`mcp_servers/`):
Built-in MCP servers that ship with the app: memory_server, email_server, image_gen_server, rag_server. These are Python stdio MCP servers using the `mcp` library.

**Companion** (`companion/`):
Thin bridge for LAN client pairing. A phone can discover the server, pair via token, and use it as a "headless brain" without duplicating LLM logic.

### Frontend Architecture (Vanilla JS SPA)

The frontend is a single-page application with no framework — pure vanilla JavaScript with a module-like file organization. Key files:

- `static/index.html` — Main SPA shell
- `static/app.js` — Application entry point
- `static/js/chat.js` (4948 lines) — Core chat UI
- `static/js/document.js` (9736 lines) — Multi-tab document editor with syntax highlighting, AI suggestions
- `static/js/slashCommands.js` (6207 lines) — Slash command autocomplete system
- `static/js/settings.js` (5094 lines) — Settings panel
- `static/js/emailLibrary.js` (5217 lines) — Email inbox UI
- `static/js/notes.js` (5109 lines) — Notes and todos
- `static/js/cookbookRunning.js` (3711 lines) — Model serving UI
- `static/js/calendar.js` (3485 lines) — Calendar UI
- `static/js/chatRenderer.js` (2356 lines) — Markdown/chat rendering
- `static/js/sessions.js` (3115 lines) — Session/chats management

Notable patterns: PWA support (manifest.json, sw.js service worker), responsive design for mobile, touch gestures, installable.

## Key Techniques

### 1. Dual Tool Execution Model

The most distinctive technical choice. The agent supports TWO tool execution paths:

**Fenced code blocks** (regex-based): The LLM writes markdown code fences with tool names as language tags:
```
```bash
ls -la
```
The agent parses these via regex patterns and dispatches to tool handlers. This works with ANY model — local llama.cpp, Ollama, any API that returns text.

**Native function calling** (OpenAI tool_use): For models that support it, tools are sent as JSON schemas in the API request and the model returns structured `tool_calls`. The `tool_schemas.py` module (1358 lines) defines the function schemas, and `function_call_to_tool_block()` converts native calls back to the internal ToolBlock format.

The system detects which mode to use based on the model and endpoint. Both paths converge on the same `execute_tool_block()` dispatcher. This is superior to most frameworks which lock you into one approach.

### 2. Dead-Host Circuit Breaker

`llm_core.py` implements an in-memory circuit breaker for upstream LLM hosts. When a connection fails:
- Counts consecutive failures per host
- After 2 consecutive failures → marks host dead for 20 seconds
- Any success resets the failure counter immediately
- Thread-safe via `threading.Lock()` (necessary because sync and async callers share the state)

This prevents one misconfigured or crashed local model from jamming all chat requests across the app. The 2-failure threshold prevents transient blips from triggering cooldowns (explicitly called out as a fix for issue #659).

### 3. Hybrid Memory Retrieval (BM25 + Vector)

`chat_processor.py` implements a two-tier retrieval system:
1. **BM25-style keyword scoring**: Tokenizes query and memories into content words (stopwords removed, min 3 chars), computes IDF across the memory corpus, scores by TF-IDF with configurable weights
2. **Optional vector similarity**: If ChromaDB is healthy, runs embedding-based similarity in parallel
3. **Recency tiebreaking**: Within the same score bucket, newer memories rank higher

The RAG similarity threshold is 0.35 — only memories scoring above this are injected into context. This hybrid approach is more pragmatic than pure embedding search, which can miss exact keyword matches.

### 4. Context Budget Adaptivity

`context_budget.py` computes an effective input-token budget:
- If the user explicitly set `agent_input_token_budget` → use it exactly (clamped to model's context window)
- Otherwise → scale to 85% of the model's discovered context window, capped at 200K
- When the window is unknown → fall back to the 6K default

This means a 128K context model gets ~109K budget automatically, while a 4K model stays at 3.4K. The user doesn't need to know their model's context size.

### 5. Cursor-Style Context Compaction

When approaching 85% of the context window, `context_compactor.py` triggers a self-summarization pass. The compaction prompt is remarkably well-designed:
- **User Goal**: One sentence
- **What Was Done**: Bullet points with specific file paths, function names, URLs, config values
- **Current State**: System state, last thing discussed
- **Pending / Next Steps**: What remains, open questions
- **Key Context**: Constraints, preferences, model names, ports, paths, versions

The compactor also sanitizes messages: drops orphaned `tool` messages (where the parent `tool_calls` was trimmed) to prevent OpenAI API errors. For small-context models (≤8K), it applies aggressive trimming up front.

### 6. Prompt Engineering as System Design

The agent preamble and rules (in `agent_loop.py`) represent ~600 lines of carefully engineered instruction. This isn't just a system prompt — it's a complete operational manual for the LLM covering:

- Tool usage protocols and anti-patterns ("NEVER use bash to create files")
- Email UID handling (distinguishing IMAP UIDs from display row numbers)
- Cookbook procedures (model serving lifecycle)
- UI conventions (clickable markdown link format for entity references)
- Error recovery patterns ("AFTER A TOOL FAILS, DO NOT GO SILENT")
- Multi-account routing rules
- Calendar workflow requirements (list calendars before CRUD)
- Background job patterns (#!bg prefix for long-running shell commands)

The sheer density of operational knowledge encoded in this prompt is what makes the agent actually useful rather than theoretically capable. It's a clear example of "the prompt IS the product."

### 7. Skill Extraction from Conversations

`services/memory/skill_extractor.py` uses an LLM to extract reusable skills from conversations. Skills are stored as markdown files with a specific format (metadata + procedure) and loaded as few-shot examples in future conversations. The skill format includes:
- YAML-like frontmatter (name, description, tags)
- Step-by-step procedures with tool blocks
- Context about when to apply the skill

This creates a self-improving loop: the agent gets better at specific tasks the more the user does them.

### 8. Workspace Confinement for Tool Security

`tool_execution.py` implements path resolution that confines file operations to allowed directories. When a workspace is set, all paths are resolved relative to it with validation. Tool security also includes blocked-tool lists per owner and admin-only tool gating.

## Design Decisions & Trade-offs

### Monolith Over Microservices
Everything runs in one Python process: web server, agent loop, task scheduler, email pollers, MCP client connections. This makes deployment trivial (`docker compose up` or `python -m uvicorn`) but means a single crash takes down everything. For a personal self-hosted tool, this is the right call — the operational simplicity matters more than horizontal scaling.

### Prompt-Based Tools Over Native Function Calling (by default)
The fenced-code-block approach consumes more context tokens (the tool descriptions are in the system prompt) and is less reliable (regex parsing can fail on malformed output), but it works with ANY model. Native function calling is supported as an upgrade path for capable models. This is the pragmatic choice for a tool that targets local/community models.

### JSON Files + SQLite Over a "Real" Database
Session data and memory are stored as JSON files (`data/sessions.json`, `data/memory.json`) while structured data (users, tokens, MCP servers, calendar, email accounts) uses SQLite via SQLAlchemy. The JSON files are simple to inspect and backup but aren't great for concurrent access. SQLite is a solid choice for single-user/small-team deployments.

### No Auth By Default
The app creates an admin account on first run and prints a password to the terminal. This is frictionless for getting started but means anyone with network access to the port can use it if you bind to 0.0.0.0. The README explicitly warns about this.

### Massive Agent Prompt vs Slim Prompt
The ~600-line agent rules make the agent dramatically more capable but consume significant context budget — a problem for 4K/8K local models. The ROADMAP acknowledges this: "Agent prompt/context bloat. Agent mode is too heavy for smaller local models." The context budget adaptivity partially mitigates this.

### Vanilla JS Over a Framework
The frontend uses no React, Vue, or Svelte — just vanilla JavaScript organized into files by feature. With ~96K lines of JS, this is a significant codebase to maintain without a framework's structure. The ROADMAP notes CSS is "basically Calypso's island" — acknowledging the maintenance challenge. But for a project that values simplicity and zero build steps, this is a defensible choice.

## Comparison to Related Projects

### vs Open WebUI
Open WebUI is the most popular self-hosted chat UI. Odysseus is more ambitious: it includes agent tool use, deep research, document editing, calendar, and email — features Open WebUI delegates to plugins or doesn't have. Open WebUI has a larger community and more polished UI. Odysseus has deeper agent capabilities and a more opinionated design.

### vs PiClaw / clawdBot
Both are in the same "personal AI assistant" space. Odysseus is a web UI with a broader feature surface (email, calendar, documents). PiClaw is a single Docker container with web UI. clawdBot works across messaging platforms. Odysseus's agent is more capable (50 tool types, MCP, skills, memory) but also heavier.

### vs LibreChat
LibreChat focuses on being a ChatGPT clone with multi-model support. Odysseus goes beyond chat into a full workspace with documents, email, calendar, and agent automation. LibreChat has better multi-user support. Odysseus has deeper single-user capabilities.

### vs Claude Code / Codex / Copilot CLI
These are CLI-first coding agents. Odysseus is a web UI with a broader scope (not just coding). Its agent can do coding tasks (shell, file ops, Python) but also email triage, calendar management, research, and document editing. The tool model is similar (tools as callable blocks) but Odysseus runs in a browser, not a terminal.

## Project Maturity Assessment

**Strengths:**
- Comprehensive test suite (400+ test files in tests/)
- Thoughtful error handling throughout (circuit breakers, retries, cooldowns)
- Security-conscious (workspace confinement, tool security, prompt injection tests)
- Cross-platform (Linux, macOS, Windows, Docker)
- Well-documented agent behavior (the prompts are documentation)

**Weaknesses:**
- Frontend code quality acknowledged as problematic (CSS "Calypso's island")
- Context bloat for small local models
- No proper multi-user database — JSON files for core data
- Cookbook reliability varies across hardware/GPU configurations
- Integration points (CalDAV, email providers) need more documentation

**Bottom line**: This is an extraordinarily ambitious personal project that punches well above its weight class. The agent architecture — particularly the dual tool execution model, context budget adaptivity, and prompt-engineering-as-system-design approach — contains genuinely novel ideas. It's rough around the edges but coherent at the core.

## Source Files by Size

| File | Lines | Purpose |
|------|-------|---------|
| src/tool_implementations.py | 4457 | All tool execution functions |
| routes/email_routes.py | 3216 | Email API endpoints |
| src/agent_loop.py | 2493 | Streaming agent loop + prompt |
| routes/cookbook_routes.py | 2329 | Model serving API |
| src/task_scheduler.py | 2286 | Cron task scheduler |
| src/builtin_actions.py | 2235 | Pre-built automation actions |
| routes/model_routes.py | 2091 | Model management API |
| core/database.py | 1979 | SQLAlchemy ORM models |
| src/visual_report.py | 1918 | Research report rendering |
| static/js/document.js | 9736 | Document editor UI |
| static/js/slashCommands.js | 6207 | Slash command system |
| static/js/settings.js | 5094 | Settings panel UI |
