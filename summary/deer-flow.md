---
url: https://github.com/bytedance/deer-flow
title: DeerFlow — ByteDance's LangGraph-based AI Super-Agent
author: ByteDance
date_fetched: 2026-07-03
date_published: 2025
---

# DeerFlow — Raw Analysis

## Source

Full clone of `https://github.com/bytedance/deer-flow` (main branch), 670 Python files / ~175K lines of Python backend code, plus a Next.js frontend and Docker infrastructure.

## Project Overview

DeerFlow is ByteDance's open-source AI super-agent system. It's a full-stack framework: a LangGraph-based agent backend with sandboxed execution, persistent memory, subagent delegation, MCP tool integration, and a Next.js chat frontend. External IM platforms (Feishu, Slack, Telegram, Discord, DingTalk, WeCom) bridge into the same agent through a FastAPI Gateway.

The repo is a monorepo split into:
- **Harness** (`backend/packages/harness/deerflow/`) — The publishable agent framework package (`deerflow-harness`), containing all agent orchestration, tools, sandbox, models, MCP, skills, config
- **App** (`backend/app/`) — The FastAPI Gateway API and IM channel integrations
- **Frontend** (`frontend/`) — Next.js App Router chat UI

Strict dependency direction: App imports Harness, never the reverse. Enforced by CI test.

## Architecture Deep Dive

### Agent Graph

The agent is built with LangChain's `create_agent()` (not raw LangGraph StateGraph), using `ThreadState` which extends `AgentState` with custom fields: `sandbox`, `thread_data`, `title`, `artifacts`, `todos`, `uploaded_files`, `viewed_images`, `promoted`, `delegations`, `skill_context`, `summary_text`. Custom reducers handle merge/deduplication semantics for each field.

Entry point: `make_lead_agent(config: RunnableConfig)` registered in `langgraph.json` as `deerflow.agents:make_lead_agent`.

### Middleware Chain (27 Middlewares)

This is the defining architectural pattern. The agent's behavior is composed through an ordered chain of middleware — each a LangChain `AgentMiddleware` subclass. The chain is built in two parts:

**Shared runtime base** (used by both lead agent and subagents):
1. `InputSanitizationMiddleware` — outermost `wrap_model_call`, sanitizes before any other middleware sees messages
2. `ToolOutputBudgetMiddleware` — caps tool output size
3. `ThreadDataMiddleware` — creates per-thread directories (`users/{user_id}/threads/{thread_id}/user-data/{workspace,uploads,outputs}`)
4. `UploadsMiddleware` — tracks newly uploaded files (lead agent only)
5. `SandboxMiddleware` — acquires sandbox, stores `sandbox_id` in state
6. `DanglingToolCallMiddleware` — injects placeholder ToolMessages for interrupted tool calls
7. `LLMErrorHandlingMiddleware` — normalizes provider failures into recoverable assistant-facing errors
8. `GuardrailMiddleware` — optional pre-tool-call authorization via pluggable provider
9. `SandboxAuditMiddleware` — security logging for shell/file operations
10. `ToolErrorHandlingMiddleware` — converts tool exceptions into error ToolMessages

**Lead-only middlewares** (appended after the base):
11. `DynamicContextMiddleware` — injects current date/memory as `<system-reminder>` into first HumanMessage, keeping the system prompt fully static for prefix-cache reuse
12. `SkillActivationMiddleware` — detects `/skill-name task` syntax and injects the SKILL.md body
13. `DurableContextMiddleware` — captures delegations and skill references before summarization can compact them, projects durable context into model calls
14. `SummarizationMiddleware` — context reduction when approaching token limits (optional)
15. `TodoListMiddleware` — plan mode with `write_todos` tool (optional)
16. `TokenUsageMiddleware` — records token usage, merges subagent usage back by message position
17. `TitleMiddleware` — auto-generates thread title after first exchange. Handles interrupted runs with fallback logic
18. `MemoryMiddleware` — queues conversations for async memory update
19. `ViewImageMiddleware` — injects base64 image data before LLM call (vision models only)
20. `DeferredToolFilterMiddleware` — hides MCP tool schemas until `tool_search` promotes them
21. `SystemMessageCoalescingMiddleware` — merges SystemMessages into a single leading one for strict backends
22. `SubagentLimitMiddleware` — truncates excess `task` tool calls to enforce concurrency limit
23. `LoopDetectionMiddleware` — detects repeated tool-call loops and forces a final answer
24. `TokenBudgetMiddleware` — enforces per-run token limits (optional)
25. Custom middlewares slot (optional)
26. `SafetyFinishReasonMiddleware` — suppresses tool execution on `finish_reason=content_filter`
27. `ClarificationMiddleware` — intercepts `ask_clarification` calls and interrupts via `Command(goto=END)` (always last)

### Tool System

Tools are assembled from four sources:
1. **Config-defined tools** — resolved from `config.yaml` via `resolve_variable()` (reflection-based dynamic import)
2. **MCP tools** — from enabled MCP servers (lazy initialized, cached with mtime invalidation)
3. **Built-in tools** — `present_files`, `ask_clarification`, `view_image`, `task` (subagent delegation), `setup_agent`/`update_agent` (custom agent persistence), `skill_manage`
4. **Community tools** — web search (Tavily, Brave, Serper, SearXNG, DDG), web fetch/scrape (Jina AI, Firecrawl, Crawl4ai, Browserless, FastCRW, InfoQuest, Exa), image search, ACP agent invocation

Tool deduplication by name prevents ambiguous schemas (issue #1803).

### Sandbox System

Abstract `Sandbox` interface with `execute_command`, `read_file`, `write_file`, `list_dir`, `glob`, `grep`, `download_file`, `update_file`. Two implementations:
- `LocalSandboxProvider` — per-thread filesystem sandboxes with virtual path mapping (`/mnt/user-data/{workspace,uploads,outputs}` → real host paths), LRU cache with 256 entries
- `AioSandboxProvider` — Docker-based isolation with warm-pool container reuse

Virtual path system translates agent-visible paths to physical storage uniformly across both providers. Bash tool has wall-clock timeout (default 600s), runs commands in their own process group, and drains background output through bounded pipe-drain threads.

### Subagent System

Dual thread pool: scheduler pool (3 workers) + execution pool (3 workers). Max 3 concurrent subagents enforced by `SubagentLimitMiddleware`.

Flow: `task()` tool → `SubagentExecutor` → background thread → poll every 5s → SSE events → result. Events: `task_started`, `task_running`, `task_completed`/`task_failed`/`task_timed_out`.

Subagent graphs are compiled with `checkpointer=False` to avoid inheriting the parent's checkpointer. Step capture persists both `AIMessage` and `ToolMessage` turns via `capture_new_step_messages` (walks the newly-appended tail of each chunk).

System prompt teaches the model a batch decomposition pattern: count sub-tasks, if N > 3 split into batches, launch only current batch, repeat until all complete, then synthesize.

### Memory System

File-based per-user memory at `{base_dir}/users/{user_id}/memory.json`. Per-agent memory at `{base_dir}/users/{user_id}/agents/{agent_name}/memory.json`.

Data structure:
- **User Context**: `workContext`, `personalContext`, `topOfMind` (1-3 sentence summaries)
- **History**: `recentMonths`, `earlierContext`, `longTermBackground`
- **Facts**: Discrete facts with `id`, `content`, `category` (preference/knowledge/context/behavior/goal), `confidence` (0-1), `createdAt`, `source`

Workflow: MemoryMiddleware filters messages → debounced queue (30s default) → background LLM extraction → atomic file write (temp file + rename) → cache invalidation. Top 15 facts injected into `<memory>` tags in system prompt.

Token counting uses tiktoken by default with a 600s retry cooldown for failed downloads; char-based estimation as fallback.

### Model Factory

`create_chat_model(name, thinking_enabled)` instantiates LLMs from config via reflection. Supports thinking toggles, vision support, vLLM-compatible endpoints (subclasses `ChatOpenAI` to preserve vLLM's non-standard `reasoning` field). Patched providers for DeepSeek, MiMo, MiniMax, StepFun. Claude provider with thinking support.

### Configuration System

Single `config.yaml` at repo root with `config_version` field for schema migration. Hot-reload: per-run fields reload on next request; infrastructure fields (database, sandbox, channels) require restart. Config values starting with `$` resolved as environment variables. MCP/skills in separate `extensions_config.json`.

### Persistence

Hybrid bootstrap strategy: empty DB → `create_all` + stamp head; legacy DB (no alembic) → create baseline tables + stamp + upgrade; versioned DB → `alembic upgrade head`. PostgreSQL uses `pg_advisory_lock` for concurrent Gateway instances; SQLite uses per-engine `asyncio.Lock`. Alembic migrations with idempotent `safe_add_column`/`safe_drop_column` helpers.

### IM Channels

Bridges external messaging platforms to the DeerFlow agent via `langgraph-sdk` HTTP client. Architecture: `MessageBus` (pub/sub hub) → `ChannelManager` (dispatcher) → platform-specific implementations. Feishu patches a running card in place; Telegram edits a "Working on it..." placeholder via `editMessageText` with channel-side throttling (1s private, 3s group); Slack/Discord use blocking `runs.wait()`.

User-owned channel connections with Telegram deep-link `/start <code>` and other platforms' `/connect <code>` flow. Single-active-owner transfer semantics enforced at DB layer by partial unique index.

## Key Design Patterns

### 1. Prefix-Cache-Optimized Prompt Architecture

The system prompt is fully static — no user-specific data, no dates. Memory and current date are injected per-turn by `DynamicContextMiddleware` as a `<system-reminder>` in the first `HumanMessage`. This lets providers cache the system prompt prefix across all users and sessions, saving latency and cost.

### 2. Durable Context Channels

`DurableContextMiddleware` solves the summarization-erasure problem: before summarization compacts conversation history, it captures task delegations and skill references from `ThreadState`. These are then projected into each model request as a hidden `HumanMessage` data block — durable across summarization boundaries without being stored as `messages` or promoted to system-role instructions (which would break prefix caching).

### 3. Deferred Tool Pattern

MCP tools can have massive schemas. Rather than bloating every request, DeerFlow hides deferred tools from the model's bound schema and only promotes them when the model explicitly calls `tool_search`. Promotion is hash-scoped and stored in `ThreadState.promoted`. This keeps the tool schema lean while still giving access to a large tool catalog.

### 4. Middleware-Driven Agent Composition

Rather than a monolithic agent loop, every behavior is a middleware. This gives clean separation of concerns: the summarizer doesn't know about memory, the subagent limiter doesn't know about tool errors. Ordering is explicit and documented. Custom middlewares plug in at a defined slot.

### 5. Batched Subagent Orchestration with Hard Cap

The system prompt teaches the model to count sub-tasks, batch them into groups of N (default 3), and only launch the current batch. `SubagentLimitMiddleware` silently truncates excess `task()` calls as a defense-in-depth measure. This prevents the model from spawning dozens of concurrent subagents.

### 6. Harness/App Split with Boundary Enforcement

The framework (`deerflow-harness`) publishes separately from the application layer (`app/`). A CI test (`test_harness_boundary.py`) enforces that harness code never imports from app. This enables the same agent to run embedded (via `DeerFlowClient`) without HTTP services.

## Code Size

- 670 Python files, ~175K lines of backend code
- Full Next.js frontend with Playwright e2e tests
- 27 middleware components
- 14+ community tool integrations
- 6 IM channel implementations
- Alembic migration system with hybrid bootstrap
- ~200 backend test files
- ~20 frontend Playwright specs
