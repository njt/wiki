---
url: https://gist.github.com/galligan/73beb12a47851dc0b3ce34aef4d8d529
date_fetched: 2026-08-07
---

Research notes from reading get-bb/bb (clone: `research/repos/get-bb/bb` in The Grid) and the locally installed `bb` 0.35.1 app.

- Repo SHA at research time: `3e3e1934f22afd38a82ccf770cd423c6c2d1c4a2`(2026-08-06)
- Installed app: `/Applications/bb.app`→ host-daemon at`…/bb-app/host-daemon/dist/`
- Primary code: `packages/agent-runtime`,`apps/host-daemon`,`apps/server`

bb does **not** call Anthropic/OpenAI HTTP APIs itself for coding threads. It shells out to **existing agent harnesses** (Codex CLI app-server, Claude Agent SDK → Claude Code CLI, Pi SDK, ACP agents like Cursor's `agent acp`) and normalizes their traffic into a shared bb thread-event model.

Two sanctioned adapter shapes:

- **In-process protocol adapter**— spawn a provider that already speaks a stable JSON-RPC wire protocol (Codex- `app-server`).
- **Bridge-process adapter**— spawn a Node bridge (- `bb-*-bridge.mjs`) that hosts an SDK / ACP client and exposes bb's common JSON-RPC surface on stdio.

Architecture path for a turn:

```
UI / CLI / HTTP API
  → bb Server (SQLite state, product policy)
  → Host daemon WebSocket RPC (thread.start / turn.submit)
  → AgentRuntime (process lifecycle, JSON-RPC framing)
  → ProviderAdapter (buildCommandPlan + translateEvent)
  → Provider process (codex app-server | bridge → SDK/ACP)
  → events stream back → daemon → server → clients
```
| Piece | Role | 
|---|---|
| Server(`apps/server`) | Source of truth. Threads, environments, providers policy. Sends host RPC commands over the daemon WebSocket. | 
| Host daemon(`apps/host-daemon`) | Provisions workspaces/worktrees, owns `createAgentRuntime()`, spawns provider processes, posts event batches. | 
| Agent runtime(`packages/agent-runtime`) | Multiplexes providers/threads; never interprets provider-specific wire content itself — adapters own that. | 
| Agent providers catalog(`packages/agent-providers`) | Built-in provider IDs, capabilities, composer actions, reasoning ladders. | 
| CLI / App | First-class clients of the same server contract. | 

Environment variables injected into agent shells include `BB_PROJECT_ID`, `BB_THREAD_ID`, `BB_ENVIRONMENT_ID`, `BB_CLI`.

From a live `bb provider list` on this machine (and the repo catalog):

| Provider ID | Display | Shape | What actually runs | 
|---|---|---|---|
| `codex` | Codex | Direct JSON-RPC | `codex app-server` | 
| `claude-code` | Claude Code | Bridge | `node bb-claude-code-bridge.mjs`→`@anthropic-ai/claude-agent-sdk`→ host`claude`CLI | 
| `pi` | Pi | Bridge | `node bb-pi-bridge.mjs`→ Pi coding-agent SDK | 
| `acp-cursor` | Cursor | Bridge + ACP child | `node bb-acp-bridge.mjs`→ spawns`agent acp` | 
| `acp-opencode` | opencode | Known ACP (if CLI on PATH) | `opencode acp` | 
| `acp-omp` | omp | Known ACP | `omp acp` | 
| `acp-grok` | Grok Build | Known ACP | `grok agent stdio` | 
| `acp-hermes-agent` | Hermes Agent | Known ACP | `hermes acp` | 
| `acp-<slug>` | Custom | Configured ACP | `~/.bb/config.json`→`customAcpAgents` | 

Built-in ACP profile in agent-runtime today is Cursor (`ACP_AGENT_PROFILES`). Other known ACP agents are registered server-side in `apps/server/src/services/system/known-acp-agents.ts` and become available when their executable is detected on the host.

`AgentRuntime` keys provider processes via `resolveProviderProcessKey`:

- **Codex live threads**:- **thread-scoped**. Process key- `codex\0thread:<bbThreadId>`→ one- `codex app-server`per bb thread.
- **Claude Code / Pi / ACP bridges**:- **provider-scoped**(process key = provider id, plus ACP launch-spec fingerprint when present). One bridge process can host multiple bb threads; ACP still spawns- **one agent child process per thread**inside the bridge.
- Maintenance probes (model list, etc.) can use a provider-scoped Codex process without a thread id.

Packaged bridge bundles live next to the host-daemon binary:

- `bb-claude-code-bridge.mjs`
- `bb-pi-bridge.mjs`
- `bb-acp-bridge.mjs`

```
codex app-server
```
Adapter: `packages/agent-runtime/src/codex/adapter.ts`

- No Node bridge. bb speaks Codex's app-server JSON-RPC over stdin/stdout.
- Types are generated from the local Codex binary (`codex app-server generate-ts`) into`packages/agent-runtime/src/codex/generated/codex-app-server/`.
- Docs pointer: `docs/codex-app-server.md`.

| bb command | Codex method | 
|---|---|
| initialize | `initialize`(clientInfo name`"bb"`,`experimentalApi: true`) | 
| model/list | `model/list` | 
| skills | `skills/extraRoots/set` | 
| thread/start | `thread/start` | 
| thread/resume | `thread/resume` | 
| thread/fork | `thread/fork` | 
| turn/start | `turn/start`(or`thread/compact/start`for standalone compact) | 
| turn/steer | `turn/steer` | 
| rename/archive | `thread/name/set`,`thread/archive`, … | 

Thread start enables `experimentalRawEvents: true` so bb can translate richer item streams. Events are normalized in `codex/event-translation.ts`.

Via Codex `config` on start/resume/turn:

- `shell_environment_policy.set.BB_THREAD_ID`
- `model_reasoning_effort`from bb reasoning level
- `memories.use_memories`/- `memories.generate_memories`(provider Settings preference)
- When native subagents disabled: `features.multi_agent=false`and V2 concurrent threads capped to`1`
- Workspace sandbox writable roots when permission scope is workspace
- Approval policy / sandbox mode mapped from bb permission modes (`accept-edits`/`auto`/`full`)

Auth: Codex's own credentials (CLI login / env). bb does not proxy OpenAI tokens for coding turns; it may reuse Codex credentials for helper inference routes separately.

```
node bb-claude-code-bridge.mjs
```
Bridge source: `packages/agent-runtime/src/claude-code/bridge/`

The bridge is a thin JSON-RPC shell. It does **not** translate to bb thread events; it forwards raw Claude Agent SDK messages as:

`{ "method": "sdk/message", "params": { "threadId": "…", "message": /* SDKMessage */ } }`The parent adapter (`claude-code/adapter.ts`) translates `SDKMessage` → `ThreadEvent[]`.

`SdkSession` (`bridge/sdk-session.ts`) calls:

```
query({
  prompt: asyncIterableOfUserMessages,
  options: { cwd, systemPrompt, model, permissionMode, sandbox, hooks, mcpServers, … }
})
```
from `@anthropic-ai/claude-agent-sdk`.

User turns are pushed into that async iterable (`pushInput`), not sent as one-shot CLI argv prompts.

The SDK is pointed at the host Claude Code executable via:

- `BB_CLAUDE_CODE_EXECUTABLE`if set and executable, else
- `claude`found on- `PATH`

(`resolveClaudeCodeExecutable` in `session-options.ts`)

So the request path is:

```
bb Server → daemon → AgentRuntime JSON-RPC
  → Claude bridge (thread/start, turn/start, …)
  → Claude Agent SDK query()
  → Claude Code CLI process (pathToClaudeCodeExecutable)
  → Anthropic (whatever the CLI/SDK uses for the selected model)
```
- System prompt: preset `claude_code`with optional append, or full replace for manager/instructionMode
- `settingSources: ["user", "project", "local"]`— loads- `~/.claude`+ project settings
- `persistSession: true`, resume via SDK- `resume`/- `forkSession`
- Permission modes mapped into Claude's (`default`,`acceptEdits`,`bypassPermissions`, …)
- Workspace sandbox via SDK `sandbox`for accept-edits/auto + workspace scope
- Readonly PreToolUse hooks for plan/readonly-style modes
- Reasoning → SDK `effort`+ adaptive thinking;`ultracode`= effort`xhigh`+ settings flag`ultracode`
- Memory: `settings.autoMemoryEnabled`
- Workflows: `settings.enableWorkflows`
- Skills injected as **local plugins**(not SDK skills allowlist — that would hide user skills)
- Dynamic bb/plugin tools exposed as an in-process MCP server named `bb-bridge`(`tool-proxy-mcp.ts`), tool names`mcp__bb-bridge__<name>`
- Native Task tool can be removed via `disallowedTools`when provider-native subagents are disabled in Settings

Interactive permissions/user questions are bridged back to bb as pending interactions, then answered via JSON-RPC responses into the SDK `canUseTool` / question handlers.

Same bridge pattern as Claude Code:

```
node bb-pi-bridge.mjs → @earendil-works/pi-coding-agent (and pi-ai)
```
Adapter owns event translation from Pi `AgentSessionEvent`s. Process is provider-scoped like Claude. Capabilities are narrower (e.g. only `full` permission mode in the live provider list).

```
node bb-acp-bridge.mjs
  └─ per thread: spawn(<agentCommand>) speaking Agent Client Protocol over NDJSON stdio
```
Bridge source: `packages/agent-runtime/src/acp/bridge/`

bb implements a **minimal ACP client** (no external ACP SDK). Frames are newline-delimited JSON-RPC. Schemas live in `acp/wire.ts`.

Built-in profile (`acp/profiles.ts`):

```
agentCommand: { command: "agent", args: ["acp"] }
modelCli: { listArgs: ["--list-models"], selectFlag: "--model", primaryModels: […] }
```
Important comment in-repo: the Cursor **editor** binary (`cursor`) is not the ACP agent; the CLI agent binary is `agent`.

Model/reasoning for Cursor-style agents is often encoded in **model id variants** selected at launch (`--model …`), not only via ACP config options. Hermes uses ACP-native `reasoning_effort` config instead.

- Spawn agent child (`agent acp`,`opencode acp`, …)
- `initialize`(protocolVersion, clientInfo- `bb`, fs capabilities)
- Authenticate if agent advertises auth methods
- `session/new`(or- `session/load`on resume when supported)
- Optional model/thought-level config via `session/set_config_option`or CLI flags
- Turns: `session/prompt`with content blocks
- Stream: `session/update`notifications (message chunks, thoughts, tool calls, plans)
- Permissions: agent → `session/request_permission`→ bridge policy / bb UI
- Client FS: agent may request `fs/read_text_file`/`fs/write_text_file`; bridge enforces workspace write roots
- Cancel: `session/cancel`
- Dynamic tools: MCP server config attached at session create (similar proxy idea as Claude)

Adapter translates ACP updates into bb `ThreadEvent`s (`acp/adapter.ts`).

- User/agent sends a message via UI, `bb thread tell`, or spawn prompt.
- Server decides `thread.start`(cold) vs`turn.submit`(warm), builds a`HostDaemonCommand`with providerId, model, permissionMode, instructions, dynamicTools, optional`acpLaunchSpec`.
- Daemon `command-handlers/thread.ts`stages attachments, resolves workspace, calls`runtime.startThread`/`runtime.runTurn`/`runtime.steerTurn`.
- Runtime ensures the provider process, sends adapter-built JSON-RPC, stamps events with bb `threadId`+`providerThreadId`.
- Events post back to the server; WebSocket clients update the timeline.
- Parent/child thread orchestration, skills injection, and plugin tools are server-owned and re-resolved at start/submit time.

Separate from provider-native files (`CLAUDE.md`, repo-root `AGENTS.md` for Codex):

- `~/.bb/AGENTS.md`— global bb instructions appended at provider session start
- `<workspace>/.bb/AGENTS.md`— workspace instructions
- bb skills under data-dir / workspace `.bb/skills/`— injected per thread
- Plugin dynamic tools registered via plugin API → become `dynamicTools`on start/submit

Provider-native memory (Codex memories, Claude auto-memory) is controlled on Settings → Providers pages and applied at start/resume/fork — not mid-turn.

- It is not a thin OpenAI/Anthropic chat client for coding agents.
- It does not require a shared bridge skeleton for every SDK (explicitly rejected; Claude/Pi bridges stay separate).
- ACP support is a **subset**of the protocol — enough for session/prompt/tools/permissions/fs used in practice.
- Custom ACP agents need a co-located daemon from the same bb version.

| Topic | Path | 
|---|---|
| Runtime public API / architecture | `packages/agent-runtime/README.md` | 
| Provider registry | `packages/agent-runtime/src/provider-registry.ts` | 
| Codex adapter | `packages/agent-runtime/src/codex/adapter.ts` | 
| Claude adapter + bridge | `packages/agent-runtime/src/claude-code/` | 
| ACP adapter + bridge | `packages/agent-runtime/src/acp/` | 
| Known ACP agents | `apps/server/src/services/system/known-acp-agents.ts` | 
| Daemon thread handlers | `apps/host-daemon/src/command-handlers/thread.ts` | 
| Runtime manager | `apps/host-daemon/src/runtime-manager.ts` | 
| System overview | `docs/system-overview.md` | 
| Config / custom ACP | `docs/configuration.md` | 

Think of bb as an **orchestrator + normalizer**:

- **Orchestrator**: projects, environments/worktrees, parent/child threads, permissions UX, skills/plugins.
- **Normalizer**: each harness keeps its own session identity and tooling; bb adapts them into one timeline and one CLI/API.

If you ask "how does bb send a request to Claude Code?", the accurate answer is: **JSON-RPC into a Node bridge that drives the Claude Agent SDK's  query() against the local claude binary** — not a direct Anthropic Messages API call from bb itself.

If you ask the same about Codex: **JSON-RPC into a per-thread  codex app-server process**, using Codex's native 

`thread/*` and `turn/*` methods.If you ask about Cursor: **JSON-RPC into the ACP bridge, which spawns  agent acp and calls ACP session/prompt.**
