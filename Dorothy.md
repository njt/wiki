# Dorothy

Free, open-source (MIT) Electron desktop app for orchestrating multiple Claude Code, Codex, Gemini, and Ollama agents simultaneously. Built by Charlie85270. Five MCP servers expose 40+ tools for programmatic control — agent lifecycle, Kanban task management, persistent shared memory (SQLite FTS5), Telegram/Slack remote control, and Twitter data. Automations poll GitHub and JIRA on cron schedules, spawning temporary agents with template-injected context. The Kanban system auto-assigns tasks to agents by skill, including creating new agents when no match exists — forming a closed loop from external issue tracking to AI execution.

---

## Key Quotes

> "Your AI Agents, Perfectly Managed"

The pitch is containment and visibility — agents as a manageable fleet, not a firehose of terminal tabs.

> "Run 10+ agents simultaneously across different projects and codebases"

This isn't hypothetical. Dorothy's node-pty-per-agent architecture means each agent gets a real terminal session with independent model selection, skill assignments, and project context. Real-time output streams side by side.

> AI agents are powerful but they can quickly become overwhelming. Dorothy keeps everything under control.

The marketing copy captures a genuine pain point. Anyone who's juggled three Claude Code sessions and lost track of which one is doing what will recognize the problem. Whether Dorothy solves it better than tmux is the real question.

> "Made with ♥ for the AI agents community"

Open-source, free forever, no account required. This isn't a SaaS growth hack — it's a community project.

---

## Architecture

Dorothy runs three layers:

1. **Electron shell** (React 19, Next.js 16) — the UI with dashboards, Kanban board, automations config, and the multi-agent terminal view
2. **Five MCP servers** (stdio-based) — the entire control surface. The Super Agent and any MCP-compatible client can drive the system programmatically
3. **Agent execution layer** — isolated Claude CLI processes via node-pty, with real-time output streaming and lifecycle state tracking

The MCP-first architecture means Dorothy isn't just a GUI — it's a platform. The same MCP tools that power the React UI are available to agents themselves. An agent can spawn another agent. This is the [[Smart Models Dumb Pipes]] pattern: the UI is a dumb pipe, the MCP tools are the control plane, and the agents are the smart endpoints.

State persists to `~/.dorothy/` (agents.json, kanban-tasks.json, vault.db) and `~/.claude/` (settings, schedules, logs). Agents survive app restarts.

---

## Key Themes

#tool #agent-orchestration #multi-agent #kanban #automation #mcp #desktop-app #developer-tools

## Feature Map

### Super Agent Orchestrator
A meta-agent that spawns, monitors, and kills sub-agents. Delegates by skill/capability. Remote-controllable via Telegram and Slack — messages without a slash command go straight to the Super Agent. This is the [[Agent Orchestration]] pattern made concrete: a planner/worker model where the planner is itself an LLM.

### Automations (Event-Driven Agents)
The most interesting feature. Poll GitHub (PRs, issues, releases via `gh` CLI) or JIRA (REST API v3) on cron schedules. When new items appear, template variables (`{{title}}`, `{{url}}`, `{{body}}`, `{{repo}}`) are injected into a prompt, a temporary agent is spawned with full MCP tool access, and output is delivered to Telegram, Slack, or posted as GitHub comments. Content-hash deduplication prevents re-processing.

This is CI/CD for AI work. It's the [[The Dark Factory is a DOT File]] pattern applied to external events — the automation configuration is the DOT file, the agent is the disposable executor.

### Kanban Task Management
Columns: Backlog → Planned → Ongoing → Done. Priority levels, progress tracking, labels, skill requirements. The `kanban-automation` service continuously matches tasks to agents by skill — if no matching agent exists, it creates one. JIRA automations auto-create Kanban backlog tasks.

This is the [[Managing Agents via Kanban Boards]] pattern in production: task status transitions as the signaling mechanism between humans, external systems, and agents. Compare to [[ralph-ban]] (TUI kanban) and [[vibe-kanban]] (inline diff review). Dorothy adds the auto-assignment closed loop, which neither of those has.

### Vault (Shared Agent Memory)
Persistent document storage with SQLite FTS5 full-text search. Nested folders, cross-agent access, file attachments. Any agent can read documents created by another. This is a practical answer to the shared-memory problem in multi-agent systems — not as sophisticated as [[Agent Memory and Context]] taxonomies or [[NornicDB]]'s temporal graph, but pragmatic: files plus full-text search gets you 80% of the way.

### Remote Control (Telegram + Slack)
Full fleet management from a phone. `/status`, `/agents`, `/start_agent`, `/stop_agent`, `/ask`, `/usage`. The Slack integration uses Socket Mode — no public URL required, which simplifies self-hosted deployment. This makes Dorothy a candidate for [[Claude Code on the Go]] workflows, though the Electron app still needs to be running somewhere.

### Google Workspace Integration
Uses the [[Google Workspace CLI]] (`gws`) via MCP. Covers Gmail, Drive, Sheets, Calendar, Docs (read/write), Slides, Tasks, Chat, People, Forms, Keep. 100+ specialized skills available. This is a deep integration — not just "search my email" but full document creation and editing.

### Skills & Plugins
One-click install from skills.sh and a built-in marketplace. Categories span code intelligence (LSP for TS, Python, Rust, Go), integrations (GitHub, GitLab, Jira, Figma, Slack, Vercel), and dev workflows (commits, PR review). This is the [[2389 Plugin Marketplace]] model applied to agent capabilities rather than Claude Code plugins.

---

## Critical Analysis

**What's genuinely good:**

The MCP-first architecture is the right call. Exposing all control surfaces through MCP tools means Dorothy is composable — you could replace the Electron UI with a TUI, a web dashboard, or another agent entirely. This is what [[Building Agents for Production Systems with MCP]] advocates.

The automation closed loop (JIRA → Kanban → agent → output → JIRA comment) is the feature that separates Dorothy from terminal multiplexers. tmux doesn't auto-assign work from your issue tracker. The content-hash deduplication shows attention to the boring edge cases that kill automation pipelines.

The Vault with cross-agent FTS is the minimum viable shared memory for multi-agent systems. It's not [[Three Tier Memory]] but it doesn't need to be — a searchable document store is what agents actually use.

**What's concerning:**

The runtime stack is heavy. Electron 33 + Next.js 16 + React 19 + Zustand + Framer Motion + xterm.js is a lot of JavaScript for what's fundamentally a PTY manager with a database. The CPU and memory cost of running this alongside multiple Claude Code processes (each of which is already expensive) needs measurement. The "wife your agents need" tagline on the website continues to be cringe.

40+ MCP tools across five servers is a lot of context window consumption. Every tool description costs tokens. [[What I learned building an opinionated and minimal coding agent]] makes the case that four tools beats forty — Dorothy bets the opposite direction. The tool count is understandable given the feature surface, but there's no evidence of tool-use optimization (grouping, context-aware filtering, per-agent tool subsets).

**The real question:** Does this beat tmux + a shell script? For a solo developer running 2-3 agents, probably not — the overhead exceeds the benefit. For a team managing fleets of agents across projects with automation pipelines feeding from GitHub and JIRA, the answer shifts. Dorothy's value scales with agent count and integration surface. The comparison isn't tmux — it's [[klaw.sh]] (kubectl for agents), [[AgentsView]] (analytics), and [[Collaborator]] (desktop agent canvas). Dorothy combines all three in one app, which is either integration genius or feature bloat. Time will tell.

---

*Sources: [[raw/dorothy]]*
*Last updated: 2026-05-14*
