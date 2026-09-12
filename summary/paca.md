---
url: https://github.com/Paca-AI/paca
title: "Paca — AI-Native Open-Source Project Management"
author: Paca-AI
date_fetched: 2026-06-21
date_published: 2025
topics:
  - agent-orchestration
---

# Source Analysis: Paca (github.com/Paca-AI/paca)

## What It Is

Paca is a self-hosted, AI-native project management platform where AI agents participate as first-class Scrum teammates — not as chatbots bolted onto a task board. It's positioned as a free, open-source (Apache 2.0) alternative to Jira, Trello, ClickUp, and Monday. The v0.4.0 release (current at analysis time) adds in-app AI chat, activity diff/revert, and a plugin marketplace.

## Repository Structure

```
apps/web/          React + TanStack Start + shadcn/ui
apps/mcp/          @paca-ai/paca-mcp — MCP server npm package
services/api/      Go 1.26 + chi router — REST API, ~71K LoC
services/realtime/ Node.js/TypeScript + Socket.IO — ~815 LoC
services/ai-agent/ Python 3.12 + FastAPI + OpenHands SDK — ~2.5K LoC
skills/            12 Agent Skills for Claude Code (/paca, /paca-do, etc.)
deploy/            Docker Compose configs (dev, prod, e2e)
docs/              Architecture, guides, plugin docs
scripts/           Install scripts (shell)
```

## Architecture Deep-Dive

### API Service (Go) — services/api/

The Go API follows a clean layered architecture:

**Domain layer** (`internal/domain/`): Pure domain entities with no framework dependencies. Each domain (task, sprint, project, doc, agent, plugin, user, apikey, attachment, notification, globalrole) has its own package with entity.go, errors.go, repository.go (interface), and service.go (business logic).

**Service layer** (`internal/service/`): Implements domain repository interfaces. Features a **cached service pattern** — `NewCachedService(inner, cacheStore, ttl, log)` wraps the real service with Redis caching for frequently-read entities (projects, tasks, sprints, views, global roles).

**Transport layer** (`internal/transport/http/`): Standard HTTP triad — handlers (request parsing, auth, response), DTOs (request/response serialization), presenter (JSON response helpers). The router (`router/router.go`, ~600 lines) registers all routes with middleware chains.

**Platform layer** (`internal/platform/`):
- `authz/` — Permission-based authorization with glob-style permissions (`tasks.read`, `tasks.*`). Global roles (SUPER_ADMIN, ADMIN, MEMBER) and project roles (Product Owner, Scrum Master, Developer, Viewer) seeded at startup.
- `plugin/` — WASM runtime via wazero. The real innovation is here (see Key Techniques below).
- `cache/` — Redis wrapper with key prefixing.
- `database/` — PostgreSQL connection + migration runner.
- `messaging/` — Redis streams (for async workers) and Pub/Sub (for real-time fan-out).
- `storage/` — S3-compatible object store (MinIO default, AWS S3 supported).
- `secret/` — AES-256-GCM encryption for agent LLM API keys at rest.
- `token/` — JWT management (access + refresh tokens).

**Repository layer** (`internal/repository/postgres/`): SQL implementations of domain repository interfaces. Direct SQL queries (no ORM), using `database/sql` + `jmoiron/sqlx`.

**Workers** (`internal/worker/`): Three Valkey stream consumers — ActivityConsumer (persists task activities), DocActivityConsumer (persists doc activities), NotificationConsumer (processes notifications and triggers agent conversations).

### AI Agent Service (Python) — services/ai-agent/

**Entry point** (`src/main.py`): FastAPI app with lifespan that starts a single asyncio worker loop on startup.

**Worker** (`src/worker.py`): Reads from Valkey stream `agent:triggers` using a consumer group. Semaphore limits concurrency (`WORKER_CONCURRENCY`). Each trigger message is processed in an asyncio task. Supports `agent.stop` control messages for graceful conversation termination.

**Executor** (`src/agent/executor.py`): The core. For each conversation:
1. Builds LLM config from agent DB record (supports any LiteLLM-compatible provider)
2. Converts DB skill rows into OpenHands SDK Skill objects with KeywordTriggers
3. Builds MCP config — user-configured servers + always-appended Paca MCP server
4. Constructs system prompt suffix from trigger type (task_assigned, comment_mention, chat_message, description_write), agent's custom system prompt, project context injection, documentation workflow instructions, and repo access workflow
5. Spins up an ephemeral Docker container running the OpenHands agent server
6. Creates an OpenHands Agent with LLM + skills + MCP + optional repo tools
7. Sends initial message via `conversation.send_message()`
8. Runs non-blocking, polls for completion or stop signal
9. Event callbacks persist to PostgreSQL and publish via Valkey for real-time UI updates

**Docker sandbox** (`src/agent/docker_workspace.py`): 
- Port pool (configurable range) for local dev port mapping
- ARM64/AMD64 platform detection
- Inside Docker: joins the same network as the ai-agent service so sandbox can reach `api` and `gateway` hostnames
- Outside Docker: maps to localhost port
- **Volume sharing**: Mounts the ai-agent source tree into the sandbox so `repo_tools.py` can be imported by the remote agent server
- **Repo tools injection**: For production (where /app is baked into the image, not bind-mounted), copies `repo_tools.py` into the container via `put_archive` before the server starts

**Repository tools** (`src/agent/repo_tools.py`): Four custom OpenHands tools:
1. `list_repositories` — Queries plugin APIs for linked repos
2. `clone_repository` — Clones via token-embedded URL, scrubs tokens from output
3. `push_branch` — Pushes with fresh auth token, auto-detects current branch
4. `create_pull_request` — Creates PR via plugin API, links to task

These are registered via `register_tool()` and loaded dynamically by the OpenHands server inside the sandbox via `importlib.import_module("src.agent.repo_tools")`.

**Prompt system** (`src/agent/prompt.py`): Builds the initial message from trigger data. Appends task ID, comment ID, chat session ID, and repository setup instructions (with ready-to-use `clone_repository` calls when there's exactly one repo).

**Builder** (`src/agent/builder.py`): Constructs LLM, skills, and MCP config from DB records. Handles LiteLLM provider routing — for unknown providers, prefixes with `openai/` and uses `base_url`.

### Realtime Service (TypeScript) — services/realtime/

**Auth flow**: Socket.IO middleware that accepts JWT from `handshake.auth.token` or cookie. Validates by calling `GET /api/v1/users/me/global-permissions` on the API service (no local JWT verification).

**Room model**: Two namespace-scoped rooms per project:
- `project:<projectId>:tasks` — task events
- `project:<projectId>:docs` — doc events

On `join`, fetches project permissions and joins only rooms the user can access. Permission map: `tasks.read` → tasks room, `docs.read` → docs room.

**Auto-join**: Each socket auto-joins `user:<userId>:notifications` on connect for personal notification delivery.

**Session persistence**: Session data stored in Valkey (not in-memory), enabling multi-replica deployments. Raw JWT token kept in memory only — never persisted.

### WASM Plugin Runtime — services/api/internal/platform/plugin/runtime.go

This is arguably the most technically sophisticated part of the codebase.

**Architecture**: Each plugin is compiled to a WASM module (wazero runtime). The host (Go API) provides a `paca` host module with these functions:

- `db_query` — SELECT queries (and DML with RETURNING) scoped to plugin's PostgreSQL schema
- `db_exec` — INSERT/UPDATE/DELETE (DDL/DCL blocked)
- `storage_get/set/delete` — Key-value store per plugin
- `tasks_list`, `task_get`, `project_get`, `members_list` — Read-only core data access
- `http_request_body`, `http_request_headers`, `http_caller_identity`, `http_respond` — Inbound HTTP request context
- `fetch` — Outbound HTTP to allowlisted domains (HTTPS only, SSRF-protected via DNS rebinding check)
- `event_emit`, `event_subscribe` — Pub/Sub event system
- `activity_record` — Write task activity (actor/project derived from auth context, not plugin payload)
- `log` — Structured logging at debug/info/warn/error levels
- `config_get` — Read host config values (only keys declared in manifest)

**Security model**:
- Per-plugin PostgreSQL schema isolation (`SET LOCAL search_path TO plugin_data_<name>`)
- Plugin declares allowed outbound domains in manifest
- Outbound fetch validates against DNS (blocks private/internal IPs)
- 50 MiB response body cap for outbound HTTP
- 5-second call duration limit, 64 MiB WASM memory limit
- Plugin `HandleRequest` serialized via mutex
- DDL/DCL SQL statements blocked in `db_exec`
- `activity_record` derives actor/project from auth context to prevent impersonation

**Lifecycle**: Load on startup → call `Init` if exported → `HandleRequest`/`HandleEvent` on demand → call `Shutdown` on unload.

### Claude Code Skills — skills/

12 Agent Skills following the [Agent Skills spec](https://agentskills.io/specification):

| Skill | Purpose |
|-------|---------|
| `/paca` | General task/doc/sprint ops |
| `/paca-epic` | Turn requirements into epic + child stories |
| `/paca-clarify` | Identify ambiguities in tasks/docs |
| `/paca-breakdown` | Decompose tasks into sub-tasks |
| `/paca-sprint` | Plan sprint from backlog |
| `/paca-estimate` | Estimate story points |
| `/paca-prioritize` | Score and set priorities |
| `/paca-do` | Execute a task end-to-end |
| `/paca-test` | Derive and run test cases |
| `/paca-doc` | Write/update documentation |
| `/paca-setup` | Interactive MCP connection wizard |

Each skill is a structured five-step workflow (e.g., `/paca-do`: load context → mark in progress → do the work → update and close → report) that uses MCP tools exclusively — never local files.

### Web Frontend — apps/web/

React + TanStack Start + shadcn/ui. File-based routing via `routeTree.gen.ts`. Custom hooks for real-time updates (`use-project-realtime.ts`), permissions (`use-permissions.ts`), and debounced callbacks. Feature-complete with board views, document editor, agent chat, settings, plugin marketplace, and admin panel.

### Database Schema

15 migrations covering: users, global_roles, projects, project_roles, project_members (with `member_type` for human/agent distinction), task_types, task_statuses, sprints, sprint_views (board/table/roadmap/plugin), view_task_positions, custom_field_definitions, tasks (with BlockNote JSON descriptions, story points, soft delete), files/attachments, doc_folders/documents/doc_snapshots/doc_activities, notifications, api_keys, plugins/plugin_extension_settings, agents/agent_mcp_servers/agent_skills/agent_chat_sessions/agent_conversations/agent_conversation_events.

Notable: `agents` table stores LLM config per agent (provider, model, encrypted API key, base URL, system prompt, max iterations, timeout). `agent_conversations` tracks OpenHands conversation state with trigger_type, status, repo info, and error messages.

## Key Techniques

### 1. Agent as Scrum Teammate (not chatbot)
Agents are rows in `project_members` with `member_type = 'agent'`. They appear on the Scrumban board alongside humans, can be assigned to sprints, pick up tasks, and update status in real time. The architectural distinction is that the agent is a participant in the team process, not an external tool.

### 2. Trigger-Primed System Prompts
The system prompt suffix is assembled from four sources, in order: agent's custom system prompt → project context injection → documentation workflow instructions → trigger-specific prompt. Trigger types (`task_assigned`, `comment_mention`, `chat_message`, `description_write`) each get different appended instructions. This means the same agent behaves differently depending on how it was invoked.

### 3. Docker Sandbox per Conversation
Every agent conversation gets its own ephemeral Docker container. Each container:
- Gets a fresh `OH_SECRET_KEY` (encrypted at rest, never reused)
- Inherits Git committer identity from agent config
- Shares the ai-agent source tree (for repo_tools import) and MCP build directory (for dev)
- Joins the same Docker network so it can reach `api` and `gateway` by hostname
- Is auto-removed on stop (`remove: true`)

### 4. Dual-Path Repo Tool Injection
Repo tools must run inside the sandbox (they're OpenHands tools registered via `importlib.import_module`), but the sandbox can't access the ai-agent service's config. Solution: two paths:
- **Dev/bind-mount**: Share `/app` as read-only volume
- **Production**: Copy `repo_tools.py` into the container via Docker's `put_archive` before the server starts
All Paca-API coordinates (base URL, API key) are passed as tool params, not imported from config.

### 5. Token Scrubbing in Git Output
`_scrub_token()` in `repo_tools.py` removes auth tokens from git command output before it reaches the LLM context, plus percent-encoded variants and `x-access-token:***@` URL patterns. Defense in depth against credential leakage through error messages.

### 6. WASM Plugin Sandbox with SQL Isolation
Each plugin gets its own PostgreSQL schema. All queries go through `SET LOCAL search_path TO plugin_data_<name>,public` in a transaction. Plugins can't escape their schema. DDL/DCL is blocked. For outbound HTTP, the host resolves the target domain's DNS and blocks private/internal IPs (SSRF protection).

### 7. Cached Service Pattern
Services that are read-heavy (projects, tasks, sprints, views, global roles) use a `CachedService` wrapper that checks Redis before calling the inner service. Cache TTLs are configurable per entity type.

### 8. Streaming Token Events
The OpenHands SDK's token callback captures streaming LLM chunks, accumulates them until `finish_reason` is set, then persists the complete message and publishes via Valkey Pub/Sub. This gives real-time streaming in the UI without polling.

### 9. Permission-Gated Real-Time Rooms
Socket.IO rooms are permission-gated at join time: the realtime service calls the API to get project permissions, then joins only the namespace rooms the user is allowed to see. No per-message permission checks needed — the room boundary enforces access.

### 10. Valkey as Universal Message Bus
Valkey/Redis serves three roles:
- **Cache** — TTL-based caching for read-heavy services
- **Streams** — Async work queues (activity persistence, notification processing, agent triggers) with consumer groups
- **Pub/Sub** — Real-time event fan-out to Socket.IO server → WebSocket clients

## Design Decisions

**Go for API, Python for AI agent.** The API is standard Go REST (chi router, sqlx, JWT). The AI agent is Python because OpenHands SDK is Python. This is a pragmatic polyglot choice — use the language that has the best library for the job.

**Agent bot user with fixed UUID.** `00000000-0000-0000-0000-000000000002` is seeded at startup with SUPER_ADMIN role. The AI agent service authenticates as this user via AGENT_API_KEY. This means all agent actions are attributable to a known identity, and the agent has full access without needing per-user credentials.

**MCP as the universal agent interface.** Both external AI tools (Claude Desktop, Claude Code) and internal agents (OpenHands) connect to Paca through the same MCP server. This means the same tool definitions serve both human-in-the-loop and autonomous agent workflows. The built-in Paca MCP server is always appended last in agent MCP config so user-configured servers can't override it.

**Configuration over code for workflows.** Workflows, statuses, field definitions, board layouts, sprint rules, and agent behavior are all driven by project-level configuration. No code changes needed to adapt Paca to a team's process.

**Plugin system as extension escape hatch.** Rather than building every feature into the core, Paca provides a WASM plugin system. Plugins can add custom routes, UI components, data models, and event handlers. The Plugin Marketplace lets users browse and install plugins from the UI.

**Soft deletes everywhere.** Tasks, projects, documents, project members all use `deleted_at` timestamps. Re-adding a removed member restores the row rather than inserting a new one (partial unique indexes with `WHERE deleted_at IS NULL`).

**BlockNote JSON for rich text.** Task descriptions and document content use BlockNote's JSON block format, not plain markdown. This enables structured rich text editing in the browser while keeping the data queryable as JSONB in PostgreSQL.

**Manual sort with position columns.** Board views use `view_task_positions` table with float `position` values for drag-and-drop reordering. The "manual sort algorithm" is documented in the architecture docs.

**Idempotent migrations on every startup.** All migration SQL uses `CREATE TABLE IF NOT EXISTS` and `INSERT … ON CONFLICT` so migrations can run safely on every deploy.

## Comparison Notes

**vs. Jira/Trello/ClickUp/Monday**: Paca's differentiator is AI agents as teammates, not add-ons. Jira has automation rules; Paca has agents that participate in sprint planning. The trade-off: Paca is self-hosted (you manage infrastructure) and free (no per-seat pricing). It's less feature-complete than Jira but more extensible via plugins.

**vs. Linear**: Linear is fast and design-focused but closed-source, cloud-only, and has no AI agent integration beyond basic copilot features. Paca is open-source, self-hosted, and AI-native.

**vs. Plane**: Plane is another open-source Jira alternative (also self-hosted). It's more mature in terms of traditional PM features but has no AI agent integration or WASM plugin system.

**vs. Taiga**: Taiga is open-source agile PM but has no AI integration and a more traditional architecture (Django + Angular).

**vs. OpenProject**: Similar self-hosted PM but GPL-licensed (vs. Apache 2.0), no AI agent integration, heavier stack.

**vs. the Kanban agent orchestration pattern**: Paca is the production incarnation of the pattern described in [[Agent Orchestration]] and [[Managing Agents via Kanban Boards]] — a Scrumban board as the human-agent coordination surface. Unlike research prototypes ([[ralph-ban]], [[weft]]), Paca is a fully self-hosted platform with auth, plugins, real-time sync, and multi-project support. Its contribution is making the pattern production-grade.

**vs. Fleet Supervisor**: Both use a queue-based work dispatch pattern, but [[Fleet Supervisor (sermakarevich)]] is a single-machine Python supervisor for coding agents, while Paca is a multi-service platform for Scrum teams. Paca's agent trigger system (Valkey streams → worker → Docker sandbox) is conceptually similar to Fleet's claim-and-spawn loop but designed for multi-tenant web deployment.

**vs. WASM plugin ecosystems**: Paca's WASM plugin system is more constrained than general-purpose WASM runtimes ([[Component Model 1.0]]). It's designed for safe extension of a specific application, not general computation. The capability-based permission model (manifest-declared domains, config keys, event subscriptions) is similar to the Wasm component model's dependency declaration but simpler and application-specific.

## Technical Strengths
- Clean layered Go architecture with clear domain/boundary separation
- Sophisticated Docker sandbox management with dev/prod dual paths
- WASM plugin runtime is genuinely impressive — SQL isolation, SSRF protection, capability manifests
- Real-time system is well-designed (permission-gated rooms, Valkey session persistence)
- Agent prompt assembly is thoughtful (context injection, doc-first workflow, trigger-specific priming)
- Token scrubbing shows attention to security detail

## Areas to Watch
- AI agent service is relatively thin (~2.5K LoC Python) — most intelligence delegated to OpenHands SDK
- No apparent agent evaluation framework or conversation quality metrics
- Plugin marketplace catalog is a single JSON URL — no curation or security review process described
- Agent bot user with SUPER_ADMIN is powerful — compromise of AGENT_API_KEY is catastrophic
- WASM plugin runtime is complex (~1,300 lines for runtime.go alone) — correctness bugs in memory management could crash plugins or leak data
