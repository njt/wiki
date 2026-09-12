---
url: https://github.com/nexu-io/open-design
title: "Open Design"
author: nexu-io
date_fetched: 2026-05-18
date_published: 2024-10 (initial), actively maintained
tags: [agent-orchestration, local-first, design-system, plugin-architecture, coding-agent, sse-streaming]
topics:
  - agent-architecture
---

# Open Design — Full Architectural Analysis

## What is it?

Open Design is a local-first design workspace where coding agents (Claude Code, Codex, Gemini CLI, Cursor Agent, etc.) generate design artifacts like websites, UI prototypes, slide decks, images, and videos. The user provides a brief; the agent runs a skill, tempered by a design system, and streams the result into a sandboxed preview. Everything runs locally — SQLite for state, filesystem for artifacts, and the user's own coding agent CLI for generation.

Repo stats: v0.7.0, Apache-2.0, Node 24 + pnpm 10.x monorepo with 6 apps, 8 packages, 3 tool packages. The daemon server.ts alone is 10,028 lines.

## Repo structure (key directories)

```
apps/
  daemon/       — Express server, agent spawning, SQLite, filesystem, SSE streaming, artifact storage
  web/          — Next.js 16 App Router + React 18 frontend
  desktop/      — Electron shell wrapping web + daemon
  packaged/     — Packaged Electron runtime entry
  landing-page/ — Astro marketing site
  telemetry-worker/ — Cloudflare Worker analytics
packages/
  contracts/    — Pure TS DTOs, SSE event unions, error shapes, API types
  plugin-runtime/ — Plugin manifest parser, validator, merger, resolver, skill adapter
  agui-adapter/ — AG-UI to Open Design event adapter
  registry-protocol/ — Zod schemas for plugin registry (npm-like package system)
  sidecar/      — Generic sidecar runtime
  sidecar-proto/ — Sidecar business protocol
  platform/     — OS process primitives
  contracts/src/prompts/ — LLM prompt templates (system, discovery, plugin-block, atom-block, deck-framework)
tools/
  dev/          — Local development lifecycle (sidecar-based process orchestrator)
  pack/         — macOS/Windows/Linux desktop packaging pipeline
  pr/           — PR management automation (gh wrapper with review lanes)
craft/          — Universal design craft rules (accessibility, typography, color, animation, form validation)
design-systems/ — 154 curated brand design systems (DESIGN.md files injected into agent context)
design-templates/ — 113 rendering templates (decks, image/video/audio)
skills/         — 31 design-generation skills the agent invokes
specs/
  current/      — Architecture boundaries, maintainability roadmap, critique spec, runtime adapter spec
  change/       — Dated change proposals with specs
```

## Architecture Pattern

**Local-first client-server with multi-agent runtime abstraction.**

The architecture is a three-layer split:

1. **Frontend (web/desktop)**: Next.js React app. Thin BFF/proxy layer. No direct filesystem, SQLite, or process access. Communicates with daemon via REST + SSE.

2. **Daemon**: The sole privileged local runtime. Express server on localhost. Owns:
   - SQLite (better-sqlite3) for projects, runs, plugins, critique runs
   - Filesystem (.od state, workspace, live artifacts)
   - Agent CLI spawning and lifecycle
   - SSE streaming to the frontend
   - MCP server implementations
   - Skill execution pipeline

3. **Coding Agent (external CLI)**: Any of 16 supported CLIs detected on PATH — Claude Code, Codex, Gemini CLI, OpenCode, Cursor Agent, Qwen, Qoder, Copilot, Pi, Kiro, Kilo, Vibe, DeepSeek, Devin, Kimi, Hermes. Each has an adapter definition (`apps/daemon/src/runtimes/defs/`) specifying how to build args, detect auth, list models, parse streams.

The shared boundary is purely TypeScript types (DTOs, SSE events, error codes) in `packages/contracts`. No Node APIs, no framework dependencies. This is enforced by linter guards.

## Key techniques

**1. Agent runtime abstraction layer**

The most sophisticated part of the codebase. Each of 16 agent CLIs gets a `RuntimeAgentDef` (`apps/daemon/src/runtimes/types.ts:39-77`) specifying:
- `bin` + `fallbackBins` — how to find the CLI on PATH (with shim chains for nvm/fnm/mise)
- `buildArgs` — a closure that constructs the CLI argument array from prompt, images, extra dirs, model, reasoning options. This is where per-agent quirks live (e.g., Codex needs `--sandbox danger-full-access` on Windows vs `workspace-write` on macOS/Linux due to sandbox policy differences)
- `versionArgs` / `helpArgs` — probe commands to detect CLI version and available flags (e.g., `--add-dir`, `--include-partial-messages`)
- `promptViaStdin` / `promptInputFormat` — avoids Windows ENAMETOOLONG (32KB CreateProcess limit) by piping prompt through stdin
- `streamFormat` / `eventParser` — how to chunk and parse CLI stdout (claude-stream-json, json-event-stream, etc.)
- `capabilityFlags` — parse `--help` output to detect optional features at runtime, not at compile time

Detection (`apps/daemon/src/runtimes/detection.ts`) is careful: it probes at the resolved launch path (not the PATH shim), discriminates ENONENT/EACCES/ENOTDIR (OS-rejected spawn) from exit codes 126/127 (wrapper script with missing target) from actual CLI errors. Version detection and --help probing are cached.

**2. Plugin/devloop pipeline system**

The plugin lifecycle (`apps/daemon/src/plugins/pipeline.ts`) implements a **headless scheduler with until-condition devloops**. A plugin manifest declares a pipeline of stages. Each stage can have `repeat: true` + an `until` condition expression. The scheduler:
- Walks stages sequentially
- Calls a caller-supplied `runStage()` for each iteration
- Evaluates the `until` expression against returned signals
- Capped by `OD_MAX_DEVLOOP_ITERATIONS` (default 10) to prevent infinite loops burning API quota
- Records every iteration to SQLite (`run_devloop_iterations` table) with artifact diffs, critique summaries, and token counts
- Emits SSE events (`pipeline_stage_started` / `pipeline_stage_completed`)
- Exported as an async iterator so callers stream stage events to SSE

The plugin runtime (`packages/plugin-runtime/`) handles manifest parsing (frontmatter + JSON Schema), merging (multi-source plugin composition), capability validation (8 known caps: `prompt:inject`, `fs:read`, `fs:write`, `mcp`, `subprocess`, `bash`, `network`, `connector`), and adapters for both Claude Code `skill` format and Open Design native `agent-skill` format.

**3. Critique Theater (Design Jury)**

A unique quality mechanism. Instead of single-pass generation, every artifact goes through a five-panelist Design Jury (Designer, Critic, Brand, A11y, Copy) running inside a **single CLI session** (not parallel processes — the model keeps the Designer's draft and all panelist notes in one context). The panelists role-play sequentially in one prompt.

Key design decisions:
- Scoreboard pure function (`critique/scoreboard.ts`): no I/O, consumes PanelEvents, buffers per-round, decides ship-vs-continue against configurable thresholds (default 8.0/10). Composites are weighted floats with 0.01 tolerance.
- Parser (`critique/parser.ts`): streaming tokenizer for `<PANELIST>`, `<ROUND_END>`, `<SHIP>` XML-ish blocks in CLI stdout. Handles partial chunks, malformed input, recovery.
- Orchestrator (`critique/orchestrator.ts`): wires parser + scoreboard to SSE bus + SQLite. Owns interrupt cascade, persistence, and degraded fallback (configurable: `ship_best` / `ship_last` / `fail`).
- Config is env-var driven (`OD_CRITIQUE_ENABLED`, `OD_CRITIQUE_MAX_ROUNDS`, `OD_CRITIQUE_SCORE_THRESHOLD`, etc.)

The split naming is deliberate: internal codename "Critique Theater" (code, env vars, telemetry), user-facing label "Design Jury" (i18n key `critiqueTheater.userFacingName`).

**4. Live Artifacts system**

A refreshable artifact type distinct from normal artifacts. Stored as a directory:
- `artifact.json` — BoundedJson payload (max 8 depth, 100 keys/object, 500 items/array, 16KB strings, 256KB serialized)
- `template.html` — Jinja2-style template with data placeholders
- `data.json` — cached data source
- `provenance.json` — origin tracking
- `refreshes.jsonl` — append-only refresh log
- `refresh.lock.json` — concurrency lock file
- `snapshots/` — version directory

The store (`live-artifacts/store.ts`) uses `isPathInside` and `resolveInside` for path traversal prevention. Refresh service runs refresh commands with timeouts, validates output against BoundedJson constraints, re-renders HTML.

**5. Sidecar process architecture**

The desktop app uses a sidecar IPC pattern: `packages/sidecar-proto` defines the business protocol, `packages/sidecar` is the generic runtime, and `apps/daemon/src/sidecar/server.ts` provides the daemon-side implementation. Process stamps have exactly five fields: `app`, `mode`, `namespace`, `ipc`, `source`. IPC runs via POSIX sockets at `/tmp/open-design/ipc/<namespace>/<app>.sock`.

**6. Memory system**

The daemon includes a filesystem-backed markdown memory store (`apps/daemon/src/memory.ts`) modeled after Claude Code's auto-memory. Layout: `MEMORY.md` index + per-fact `.md` files with YAML frontmatter (`name`, `description`, `type`). Types: user, feedback, project, reference. An EventEmitter bus pushes change events to SSE clients so the UI auto-refreshes. Two extraction pipelines: heuristic (regex-based) and LLM (prompt-based via `memory-extractions.ts`).

## Architecture quality assessment

**Strengths:**
- The agent abstraction layer is genuinely well-designed. Adding a new CLI takes ~150 lines of config. The probe-then-spawn pattern handles nvm/fnm/mise PATH issues correctly.
- The web/daemon boundary discipline is explicit and enforced: contracts package has zero framework dependencies, shared code is pure TypeScript, and AGENTS.md files at each level enforce this.
- The plugin pipeline with until-condition devloops is a clever way to give plugins looping behavior without giving them direct process control.
- The critique system's single-session multi-panelist approach is a pragmatic choice: avoids multi-process trace correlation and keeps model context coherent, at the cost of losing parallel panelist execution.

**Weaknesses:**
- `server.ts` at 10,028 lines is called out in the maintainability roadmap as P1 risk. The roadmap (W5) plans to split into `routes/`, `services/`, etc.
- The daemon has several `@ts-nocheck` files (server.ts, agents.ts, projects.ts, runs.ts, cli.ts) — acknowledged as deferred work.
- Test coverage is described as thin around daemon behavior (R10 in maintainability roadmap).
- SQLite migration lifecycle is "needs hardening" (R9).

## Design trade-offs

- **Simplicity over parallelism**: Single CLI session per artifact, even for multi-round critique. The critique spec explicitly rejects a parallel-process architecture, estimating 2-3 weeks additional timeline to implement cross-process artifact handoff and multi-process trace correlation.
- **Express over Fastify**: Chosen for momentum, not performance. The roadmap says "revisit only after TS, contracts, validation, tests, and modularization are in place."
- **Filesystem over database**: Artifacts, memory, and live artifacts are stored as files in `.od/`, not in SQLite. Gives users direct filesystem access at the cost of query flexibility.
- **Startup-time detection over static config**: Agent capabilities (model lists, auth status, available flags) are probed at daemon startup via CLI invocation, not stored in config files. Makes the UI self-healing (re-installs are detected) but adds startup latency.
- **TypeScript contracts over runtime schemas**: The shared boundary uses TypeScript types, not Zod schemas. The maintainability roadmap (W4) calls out this gap: "type correctness alone cannot protect against malformed runtime input."

## Comparison to related projects in this wiki

- **[[Harness Engineering]]**: Open Design's plugin pipeline + critique theater is a concrete implementation of Böckeler's feedforward/feedback framework. The `until` condition on pipeline stages is computational feedforward; the critique jury is inferential feedback.

- **[[Gas Town's Agent Patterns]]**: Open Design takes the opposite approach: hierarchical roles (the five panelists) and a single session, rather than Gas Town's unhinged orchestrator. Open Design's critique spec is explicitly anti-parallel-process, arguing for coherent model context.

- **[[Mission Control — Bhanu's 10-Agent Squad on OpenClaw]]**: Similar idea (file-based memory, multi-agent), but Open Design runs agents in one session versus Bhanu's 10 specialized agents with Convex-backed Kanban.

- **[[Scaling Long-Running Agents]]**: Cursor's finding that "flat self-coordination fails; planner/worker/judge works" aligns with Open Design's Design Jury (five fixed roles, judge-and-ship semantics).

- **[[Spec-Driven Development]]**: Open Design's design systems (154 DESIGN.md files) and craft rules are a spec layer for visual output — the design equivalent of a spec.md constraining code generation.

- **[[Claude Code is a Beast — Tips from 6 Months of Hardcore Use]]**: Open Design demonstrates a similar philosophy (skills auto-activation, plugin pipeline) but applied to design generation rather than code generation.

- **[[ctx – Agentic Development Environment]]**: Similar local-first, multi-agent architecture, but Open Design focuses on design output while ctx focuses on code output.

## Innovation points

1. **Agent-agnostic design generation**: Open Design doesn't care which CLI the user has installed. It detects 16 different coding agent CLIs and constructs the right invocation for each. This is a genuine architectural contribution — it treats coding agents as a commodity runtime layer.

2. **Scoreboard-driven artifact quality**: The critique theater's weighted composite scoring with auto-converging rounds is a specific, implementable quality mechanism. Most agent quality systems are prompt-level ("be thorough"); this is protocol-level.

3. **Plugin registry protocol**: A full npm-like package system (`packages/registry-protocol/`) for design plugins: versioning, distribution (GitHub releases, HTTPS archives, local archives), signatures (GitHub OIDC, Cosign, Minisign), yanking, publishing, doctor checks. Not just a marketplace — an actual registry protocol with Zod schemas.

4. **BoundedJson for live artifacts**: Instead of ad-hoc size limits, the live artifacts system defines explicit structural bounds (max depth, max keys, max string length, max serialized bytes) enforced at validation time.

5. **Until-condition devloop**: Plugin pipeline stages iterate with a parsed condition language (`evaluateUntil`), capped by a configurable iteration ceiling. This is a middle ground between one-shot and unbounded loops — precise enough for plugins, safe enough for production.

## Key metrics from code

- 16 agent CLI adapters (`apps/daemon/src/runtimes/defs/`)
- 31 design skills
- 154 design systems
- 113 design templates
- 12 craft rules
- ~30 API routes in the daemon
- ~38 critique/theater module files
- server.ts: 10,028 lines (P1 risk, being split)
- agents.ts: just 22 lines (thin re-export, logic in runtimes/)

## Tech stack

- TypeScript throughout (with residual `@ts-nocheck` in 5 daemon core files)
- Node 24, pnpm 10.33.2 (strict version enforcement)
- Express (daemon), Next.js 16 App Router (web), Electron (desktop)
- better-sqlite3 (zero-config local SQLite)
- Zod (registry protocol, critique config validation)
- esbuild (bundling tools)
- Nix (optional NixOS packaging via nix/)
- Helm (Kubernetes deployment for hosted version via tools/pack/helm/)
