# Paca

Self-hosted, AI-native project management platform (Apache 2.0) where AI agents participate as first-class Scrum teammates — not as chatbots bolted onto a task board. Go API, Python AI agent service (OpenHands SDK), TypeScript real-time layer, WASM plugin sandbox. v0.4.0, ~75K total LoC across 4 services.

---

## Architecture

Paca is a **4-service monorepo** connected by PostgreSQL (persistence) and Valkey/Redis (cache + message bus):

| Service | Language | Framework | Role |
|---------|----------|-----------|------|
| `services/api` | Go 1.26 | chi router | REST API, auth, plugins, ~71K LoC |
| `services/ai-agent` | Python 3.12 | FastAPI + OpenHands SDK | Agent orchestration, ~2.5K LoC |
| `services/realtime` | TypeScript | Socket.IO | Real-time event fan-out, ~815 LoC |
| `apps/web` | TypeScript | React + TanStack Start | UI |

**API layering** (`services/api/internal/`): Domain entities → repository interfaces → service implementations (with Redis caching wrapper) → HTTP handlers/DTOS → chi router with middleware. Clean separation: domain packages (`domain/task/`, `domain/sprint/`, etc.) have zero framework dependencies. Platform packages (`platform/plugin/`, `platform/authz/`, `platform/cache/`) provide cross-cutting infrastructure.

**Agent flow:** Human action (assign task, mention in comment, chat message, write description) → API publishes trigger to Valkey stream `agent:triggers` → ai-agent worker reads trigger → loads agent config from DB → builds LLM + skills + MCP + system prompt → spins up ephemeral Docker sandbox running OpenHands server → agent executes in sandbox → events streamed back via callback → persisted to PostgreSQL + published via Valkey Pub/Sub for real-time UI.

**Real-time flow:** Socket.IO clients authenticate via JWT (validated by calling API). On `join`, server fetches project permissions and joins namespace-scoped rooms (`project:<id>:tasks`, `project:<id>:docs`). Events from the API (task updates, doc changes, agent messages) flow through Valkey Pub/Sub → Socket.IO server → broadcast to permitted rooms.

**Key files:**
- `services/api/internal/bootstrap/app.go` — dependency wiring (~300 lines, manual DI)
- `services/api/internal/platform/plugin/runtime.go` — WASM plugin runtime (~1,300 lines)
- `services/ai-agent/src/agent/executor.py` — conversation executor (~560 lines)
- `services/ai-agent/src/agent/docker_workspace.py` — Docker sandbox management (~340 lines)
- `services/ai-agent/src/agent/repo_tools.py` — custom repository tools (~600 lines)
- `services/ai-agent/src/agent/builder.py` — LLM/skills/MCP construction (~105 lines)
- `services/realtime/src/server.ts` — Socket.IO server with auth and rooms (~235 lines)

## Key Techniques

### Agent as Project Member
Agents are rows in `project_members` with `member_type = 'agent'`. They appear on the Scrumban board alongside humans, can be assigned to sprints, and update task status in real time. Each agent has its own LLM config (provider, model, encrypted API key, system prompt, max iterations), MCP servers, and skills. This is architectural, not cosmetic — agents have the same data model as human members.

### Docker Sandbox per Conversation
Every agent conversation gets an ephemeral Docker container (`services/ai-agent/src/agent/docker_workspace.py`). Each container: gets a fresh encryption key; inherits Git committer identity from agent config; shares the ai-agent source tree (so `repo_tools.py` can be imported); joins the Docker network to reach `api` and `gateway` by hostname; auto-removes on stop. Port pool for local dev. Dual-path repo tool injection: bind-mount in dev, `put_archive` in production.

### Trigger-Primed System Prompts
The system prompt suffix is assembled from four sources (`services/ai-agent/src/agent/executor.py:344-438`): agent's custom system prompt → project context injection → documentation workflow instructions → trigger-specific prompt. Trigger types (`agent.task_assigned`, `agent.comment_mention`, `agent.chat_message`, `agent.description_write`) each get different appended instructions. Documentation workflow is enforced: "Before starting any task, read the project documentation."

### Custom Repository Tools
Four OpenHands tools that let agents work with linked Git repos (`services/ai-agent/src/agent/repo_tools.py`): `list_repositories`, `clone_repository` (token-embedded URL, output scrubbed), `push_branch` (fresh auth token, auto-detect branch), `create_pull_request` (links to task). Tokens in git output are scrubbed before reaching the LLM context — including percent-encoded and URL-pattern variants.

### WASM Plugin Sandbox
Plugins compile to WASM (wazero runtime) and get a `paca` host module with: SQL queries scoped to per-plugin PostgreSQL schemas (`SET LOCAL search_path TO plugin_data_<name>`); key-value storage; read-only core data access; outbound HTTP to manifest-declared domains (HTTPS only, DNS rebinding protection); event pub/sub; activity recording (actor derived from auth context, not plugin payload). DDL/DCL blocked. 5s call limit, 64 MiB memory limit, 50 MiB fetch response cap.

### MCP Everywhere
The built-in Paca MCP server (`@paca-ai/paca-mcp`) is always appended last in agent MCP config — user servers can't override it. The same server connects Claude Desktop, Claude Code skills, and internal OpenHands agents. 12 Claude Code skills (Agent Skills format) structure agent workflows via MCP tools exclusively — never local files.

### Streaming Token Callbacks
The OpenHands SDK's token callback accumulates LLM streaming chunks until `finish_reason`, then persists the complete message and publishes via Valkey for real-time UI. Event callback filters out noise (StreamingDeltaEvent, ConversationStateUpdateEvent, SystemPromptEvent) and skips agent MessageEvents (captured by token callback) to avoid duplicates.

### Permission-Gated Real-Time Rooms
Socket.IO rooms are gated at join time: fetch project permissions, join only rooms the user can access (`tasks.read` → tasks room, `docs.read` → docs room). No per-message checks. Auto-join `user:<id>:notifications` on connect. Sessions persisted in Valkey (not memory) for multi-replica support. Raw JWT in memory only.

### Configuration + Plugins Over Features
Small core (~75K LoC total). Workflows, statuses, fields, board layouts, sprint rules, agent behavior — all configuration-driven. Everything else: WASM plugins (backend) or module bundles (frontend). Plugin Marketplace in the UI for one-click install.

## Design Decisions

**Polyglot by necessity, not choice.** Go for the API (performance, type safety, ecosystem), Python for the AI agent (OpenHands SDK is Python), TypeScript for real-time and frontend (ecosystem). Each service uses the language that has the best library for its job.

**Agent bot user with fixed UUID.** `00000000-0000-0000-0000-000000000002` is seeded at startup with SUPER_ADMIN. All agent actions are attributable. Trade-off: compromise of AGENT_API_KEY is catastrophic.

**Valkey as tri-role infrastructure.** Cache (TTL-based), message broker (streams with consumer groups), and real-time bus (Pub/Sub). One Redis instance, three distinct usage patterns.

**Soft deletes everywhere.** Deleted tasks, projects, docs, and members are marked with `deleted_at` timestamps. Partial unique indexes with `WHERE deleted_at IS NULL`. Re-adding a removed member restores the row.

**BlockNote JSON for rich text.** Task descriptions and document content use BlockNote's JSON block format — structured, queryable as JSONB, but not plain markdown. Requires the BlockNote editor.

**Idempotent startup migrations.** All SQL uses `CREATE TABLE IF NOT EXISTS` and `INSERT … ON CONFLICT`. Safe to run on every deploy. Schema is the source of truth; DBML diagram is advisory.

**Manual DI over framework.** `bootstrap/app.go` wires ~50 dependencies manually. No DI framework. Verbose but transparent — you can trace every dependency without magic.

**Cached service wrapper pattern.** `NewCachedService(inner, cache, ttl, log)` wraps read-heavy services. Configurable TTLs per entity type. Simple, composable, no cache invalidation bugs from distributed state.

## Comparison Notes

**vs. Jira/Trello/ClickUp/Monday**: Paca's differentiator is AI agents as teammates, not add-ons. Self-hosted and free vs. SaaS and per-seat pricing. Less feature-complete but more extensible. The trade-off is operational burden vs. control and cost.

**vs. Linear**: Linear is faster and more polished but closed-source, cloud-only, no AI agent integration beyond copilot features. Paca is open-source, self-hosted, AI-native.

**vs. Plane**: Both are open-source Jira alternatives. Plane is more mature for traditional PM but has no AI agent integration or WASM plugins.

**vs. the kanban orchestration pattern**: Paca is the production incarnation of [[Agent Orchestration]]'s kanban-as-coordination-surface pattern. Unlike research prototypes ([[ralph-ban]], [[weft]]), it's a full platform with auth, plugins, real-time sync, multi-project support.

**vs. [[Fleet Supervisor (sermakarevich)]]**: Both use queue-based work dispatch. Fleet is a single-machine Python supervisor for coding agents; Paca is a multi-service Scrum platform. Similar Valkey-stream trigger pattern, different scope and deployment model.

**vs. [[Component Model 1.0]]**: Paca's WASM plugin system is application-specific, not general-purpose. Capability manifests are similar in spirit to Wasm component dependency declarations but simpler and more constrained.

---

#tool #project #agent-architecture

*Sources: [[summary/paca]]*
*Last updated: 2026-06-21*
