# DeerFlow

ByteDance's open-source AI super-agent platform: a LangGraph-based multi-agent system with sandboxed execution, persistent memory, subagent delegation, and a middleware-driven architecture. 175K lines of Python across 670 files, with a Next.js frontend and six IM channel integrations.

---

## Architecture

DeerFlow's backend is split into two layers with a strict dependency boundary enforced in CI:

- **Harness** (`packages/harness/deerflow/`, ~175K LOC) — The publishable agent framework. Contains all agent orchestration, middleware, tools, sandbox, MCP, skills, and model adapters. Imported as `deerflow.*`.
- **App** (`backend/app/`) — The FastAPI Gateway + IM channel integrations. Imported as `app.*`. App imports Harness; never the reverse.

The agent itself is a LangGraph graph created via `create_agent()` with a custom `ThreadState` that extends `AgentState`. The graph's entry point is `make_lead_agent(config)` at `deerflow/agents/lead_agent/agent.py:418`.

### Service Topology

Four services behind an Nginx reverse proxy on port 2026:

| Service | Port | Role |
|---------|------|------|
| Nginx | 2026 | Unified entry — proxies `/api/langgraph/*` to Gateway runtime, other `/api/*` to REST routers, everything else to frontend |
| Gateway API | 8001 | FastAPI REST + embedded LangGraph agent runtime |
| Frontend | 3000 | Next.js 16 with TanStack Query, streamdown, Shadcn UI |
| Provisioner | 8002 | Optional — Kubernetes sandbox container management |

### The Middleware Chain

This is DeerFlow's defining architectural pattern. The agent's behavior is composed from **27 middleware components** assembled in strict order. Rather than a monolithic agent loop, each capability is a `LangChain AgentMiddleware` subclass.

**Shared runtime base** (used by both lead agent and subagents):
1. `InputSanitizationMiddleware` — outermost `wrap_model_call`, sanitizes before any middleware sees messages
2. `ToolOutputBudgetMiddleware` — caps tool output size to prevent context blowout
3. `ThreadDataMiddleware` — creates per-thread directories under per-user isolation (`users/{user_id}/threads/{thread_id}/user-data/{workspace,uploads,outputs}`)
4. `UploadsMiddleware` — tracks newly uploaded files
5. `SandboxMiddleware` — acquires sandbox, stores `sandbox_id` in state
6. `DanglingToolCallMiddleware` — injects placeholder ToolMessages for interrupted tool calls
7. `LLMErrorHandlingMiddleware` — normalizes provider failures into recoverable errors
8. `GuardrailMiddleware` — optional pre-tool-call authorization
9. `SandboxAuditMiddleware` — security logging for shell/file ops
10. `ToolErrorHandlingMiddleware` — converts tool exceptions to error ToolMessages

**Lead-only middlewares** (appended after the base, at `agent.py:260`):
11. `DynamicContextMiddleware` — injects date/memory as `<system-reminder>` in first HumanMessage, keeping the system prompt fully static
12. `SkillActivationMiddleware` — detects `/skill-name task` and injects SKILL.md
13. `DurableContextMiddleware` — captures delegations/skill refs before summarization compacts them
14. `SummarizationMiddleware` — context reduction when approaching token limits
15. `TodoListMiddleware` — plan mode with `write_todos` tool
16. `TokenUsageMiddleware` — records usage, merges subagent usage by message position
17. `TitleMiddleware` — auto-generates thread title, handles interrupted runs
18. `MemoryMiddleware` — queues conversations for async memory update
19. `ViewImageMiddleware` — injects base64 images (vision models only)
20. `DeferredToolFilterMiddleware` — hides MCP tool schemas until `tool_search` promotes them
21. `SystemMessageCoalescingMiddleware` — merges SystemMessages for strict backends
22. `SubagentLimitMiddleware` — truncates excess `task` calls to enforce concurrency cap
23. `LoopDetectionMiddleware` — detects repeated tool-call loops
24. `TokenBudgetMiddleware` — per-run token limits
25. Custom middlewares slot
26. `SafetyFinishReasonMiddleware` — suppresses tool execution on content_filter
27. `ClarificationMiddleware` — intercepts `ask_clarification`, interrupts via `Command(goto=END)` (always last)

See `deerflow/agents/lead_agent/agent.py:260-393` for the full assembly.

## Key Techniques

### Prefix-Cache-Optimized Prompt Architecture

The system prompt in `deerflow/agents/lead_agent/prompt.py:364` is fully static — no user data, no dates, no thread state. Memory and current date are injected per-turn by `DynamicContextMiddleware` as a `<system-reminder>` in the first `HumanMessage`. This lets LLM providers cache the system prompt prefix across all users and sessions, saving latency and cost. This is a deliberate optimization, not an accident — the comment at `prompt.py:823-826` calls it out explicitly.

### Durable Context Channels

`DurableContextMiddleware` (`deerflow/agents/middlewares/durable_context_middleware.py`) solves a hard problem: when `SummarizationMiddleware` compacts conversation history to stay under token limits, it can erase references to ongoing subagent delegations and loaded skill files. The durable context middleware captures these before compaction and projects them into each model request as a hidden `HumanMessage` data block — they survive summarization without being stored as `messages` or promoted to system-role instructions (which would break prefix caching). The three channels are: `summary_text` (compressed history), delegation ledger (in-progress + terminal results), and skill context (name/path/description references).

### Deferred Tool Pattern

MCP tools can have enormous schemas. DeerFlow hides deferred (MCP) tool schemas from the model's bound function list and only promotes individual tools when the model calls `tool_search`. `DeferredToolFilterMiddleware` reads per-thread promotions from `ThreadState.promoted` (hash-scoped to detect catalog changes) and dynamically adds promoted tools to the model binding. This keeps the tool schema lean while still giving access to a potentially large catalog. The `tool_search` tool itself is assembled at agent construction time in `tools/builtins/tool_search.py`.

### Batched Subagent Orchestration

The system prompt (`prompt.py:236-361`) teaches the model a batch decomposition pattern: count sub-tasks, batch into groups of N (default 3), launch only the current batch, wait for results, then launch the next batch. `SubagentLimitMiddleware` silently truncates excess `task()` calls as a defense-in-depth layer. Subagents run in a dual thread pool (3 scheduler + 3 executor workers) via `subagents/executor.py`, with 5-second polling and SSE event emission. Subagent graphs are compiled with `checkpointer=False` to avoid inheriting the parent's checkpointer.

### Harness/App Split with CI Enforcement

The framework (`deerflow-harness`) is a publishable Python package; the application layer (`app/`) is an unpublished consumer. A CI test (`test_harness_boundary.py`) enforces that harness code never imports from app — the boundary is structural, not aspirational. This enables the same agent graph to run embedded (via `DeerFlowClient` at `deerflow/client.py`) without HTTP services, sharing config files and data directories.

### Provider Patching Layer

Rather than requiring all LLM providers to conform to a single interface, DeerFlow maintains a family of patched LangChain chat models at `deerflow/models/`:
- `patched_deepseek.py` — replays `reasoning_content` across tool-call turns
- `patched_mimo.py` — same reasoning replay for MiMo
- `patched_stepfun.py` — same for StepFun
- `patched_minimax.py` — strips per-message `name` field that MiniMax rejects
- `patched_openai.py` — handles OpenAI Responses API compatibility
- `vllm_provider.py` — subclasses `ChatOpenAI` to preserve vLLM's non-standard `reasoning` field
- `claude_provider.py` — OAuth + API key auth with thinking support
- `mindie_provider.py` — handles MindIE's streaming limitations
- `openai_codex_provider.py` — CLI-backed model via `codex exec`

### Memory as Structured JSON with LLM Extraction

Memory is stored as per-user JSON files with a dual-extraction model: the `MemoryUpdater` (`deerflow/agents/memory/updater.py`) uses an LLM to produce both structured facts (with `content`, `category`, `confidence`, `source`) and narrative summaries across user context and history dimensions. Facts are deduplicated by whitespace-normalized content before append. The memory queue debounces (30s default), batches updates, and writes atomically via temp-file + rename. Top 15 facts + context summaries are injected into `<memory>` tags, budgeted by token count (tiktoken with 600s retry cooldown, or char-based estimation).

### Hybrid Database Bootstrap

The persistence layer (`deerflow/persistence/bootstrap.py`) handles three database states: empty (fresh `create_all` + stamp), legacy (baseline-only `create_all` + stamp + `upgrade head`), and versioned (`upgrade head`). PostgreSQL uses `pg_advisory_lock` for concurrent Gateway instances; SQLite uses per-engine `asyncio.Lock`. Column migrations use idempotent `safe_add_column`/`safe_drop_column` helpers that no-op when the change is already present.

## Design Decisions

### Optimized for: Composability and Extensibility

The middleware chain, the reflection-based tool/model loading (`resolve_variable("module.path:ClassName")`), the MCP integration, the skill system, the ACP agent bridge — everything is designed so new capabilities can be added through config, not code. This is a platform, not a product.

### Optimized for: Production Multi-Tenancy

Per-user isolation for memory, thread data, sandboxes, and file storage. Per-user IM channel connections. Per-agent memory and config. Auth with OIDC support. The system is designed for teams, not individuals.

### Optimized for: Provider Diversity

No hard dependency on a single LLM provider. The model factory supports 15+ providers through a unified config schema with provider-specific patches for quirks. The cost: each new provider may need a custom adapter class.

### Sacrificed: Operational Simplicity

The single-worker Gateway constraint (because `RunManager` + `StreamBridge` are in-process singletons) means you can't horizontally scale the Gateway without Redis stream bridge support (not yet implemented). This is a real production limitation documented in the AGENTS.md.

### Sacrificed: Agent Predictability

The middleware chain is powerful but complex. With 27 middlewares, debugging why an agent behaved a certain way requires understanding the interaction order. The system trades transparency for capability.

## Comparison Notes

**vs. Claude Code**: DeerFlow is a *platform for building* agent applications, not a coding agent itself. It's the kind of harness Claude Code *runs on*. Both use middleware-like hooks, but DeerFlow's middleware chain is longer and more formalized. DeerFlow supports arbitrary LLM providers; Claude Code is Anthropic-only.

**vs. [[Hermes]]** (149k stars): Hermes is a personal agent framework focused on self-improving skills. DeerFlow shares the skill system concept but is designed for multi-user deployment with IM channels and auth — it's enterprise-oriented where Hermes is personal.

**vs. [[Apache Burr]]**: Burr models agents as explicit state machines with decorators on plain functions. DeerFlow uses LangGraph/LangChain's higher-level `create_agent()` abstraction with a middleware chain. Burr prioritizes debuggability through explicit state transitions; DeerFlow prioritizes composability through middleware.

**vs. [[Zeroclaw]]** (Rust, 31k stars): Both are agent runtimes with tool systems and channel integrations. Zeroclaw is Rust-native with OS-level sandboxing; DeerFlow is Python with Docker/local sandbox options. Zeroclaw uses traits; DeerFlow uses middleware.

**vs. [[Elysia]]**: Weaviate's decision-tree agent constrains tool choice per node. DeerFlow uses deferred tool loading instead — tools start hidden and get promoted on demand. Both solve the "too many tools" problem but take opposite approaches: restrict-at-design-time vs. restrict-at-runtime.

**vs. [[Slate]]**: Slate's thread-and-episode architecture routes context as a core primitive. DeerFlow's `DurableContextMiddleware` solves a similar problem (context survival across compaction boundaries) but within a single thread rather than across episodes.

**vs. [[Building Reliable Agentic AI Systems]]** (Bayer/PRINCE): Both are production agent harnesses with multi-layer safety. DeerFlow's defense-in-depth (loop detection, safety finish reason, guardrails, circuit breaker, token budgets) mirrors PRINCE's reflection loops. DeerFlow is more configurable; PRINCE is more opinionated.

## Tags

#tool #project #agents #memory #sandbox #mcp #orchestration

---
*Sources: [[raw/deer-flow]]*
*Last updated: 2026-07-03*
