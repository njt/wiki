# Lovelace

Local-first, file-based project management for agent-heavy software development. Lovelace stores every ticket, document, architecture decision, and agent session record as plain Markdown files with YAML frontmatter inside `.lovelace/` at the repo root — then provides a Tauri desktop app and a Claude Code MCP integration that treat those files as the database. No server, no cloud, no account. What Obsidian is to Notion, Lovelace is to Jira and Linear.

---

## Architecture

Lovelace is a pnpm workspace monorepo (Node 22, TypeScript strict mode) with three packages and one app:

### `packages/core` — the engine

Pure TypeScript logic with no UI or process-level IO assumptions. Everything else calls into it.

- **`frontmatter.ts`**: Parses YAML frontmatter from Markdown, preserving original line endings (LF/CRLF/CR) by slicing the body from the original text rather than re-joining split lines. Returns a `keyLines` map so validation errors can point at the exact YAML key line.
- **`config.ts`**: Loads `manifest.yaml`, `schema.yaml`, and `actors.yaml` via Zod schemas. Every schema uses `.passthrough()` — unknown keys are tolerated (ADR-0011: tolerant reads, non-destructive writes), letting newer-minor spec constructs survive through a tooling version that doesn't understand them.
- **`validate.ts`**: Full project validation — ID conventions (`T-####` format, filename match), type/status existence, field validation against runtime schema definitions, reference link integrity, session completeness (required headings), comment naming, review staleness.
- **`fields.ts`**: Schema-driven field validation. `validateTicketFields()` checks types, required fields, enum membership, and reference format against the field definitions in `schema.yaml`. `collectReferences()` gathers reference values for link integrity checking.
- **`index-gen.ts`**: Builds the deterministic `index.json` — sorted lists, fixed key order, no timestamps. Running on unchanged input produces byte-identical output. Also generates committed `BOARD.md` (tickets grouped by status, readable on GitHub).
- **`links.ts`**: Extracts the wiki-link graph from ticket and document bodies via `[[id]]` regex. Excludes `[[file:...]]` and `[[@actor]]` tokens. Deduplicated, sorted, deterministic.
- **`mutate.ts`**: All state mutations — create/update/delete tickets, log sessions, add comments, write schema/manifest/actors edits. Schema renames cascade into existing tickets. Presence management with heartbeat and GC. Per-session active-ticket markers.
- **`ids.ts`**: Sequential ID assignment using `mkdirSync` for exclusive locking (atomic on all platforms). Retries at 25ms up to 5s timeout. Missing counters are rebuilt by scanning existing entity filenames.
- **`digest.ts`**: A compact orientation summary for agent session starts — in-progress work, ready tickets, last 3 session records with open questions, validation warnings. Around 1,500 tokens.

### `packages/mcp` — the agent interface

Three processes, compiled as Bun sidecar binaries and bundled with the app so users don't need Node:

- **`server.ts`** (`lovelace-mcp`): The MCP server on stdio transport (official TypeScript MCP SDK). Exposes exactly eight tools — `create_ticket`, `update_ticket`, `describe_schema`, `query_tickets`, `read_document`, `log_session`, `set_active_ticket`, `search`. Tool descriptions are generated from the live schema at registration, including per-field value hints with enum values resolved.
- **`helper.ts`** (`lovelace-agent`): The headless helper that Claude Code hooks invoke — `digest`, `session-check`, `guard` (blocks direct edits to ticket files), `presence-start`/`presence-beat`/`presence-clear`, `track-active`. All presence operations fail open (never block a turn). Reads Claude Code's hook payload JSON from stdin to determine session context.
- **`host.ts`** (`lovelace-host`): A stateless bridge between the desktop app's Rust shell and `packages/core`. One JSON request on stdin/argv, one JSON response on stdout, process exits. Handles ~30 operations including project init/migrate/snapshot, all mutations, schema edits, file manipulation within the documentation tree, Claude/OpenCode integration install, and git commit lookups.
- **`claude.ts`**: Generates and installs Claude Code assets — CLAUDE.md marker section, `.mcp.json`, hooks in `.claude/settings.json`, slash commands (`/ticket`, `/done`), and opt-in prepare-commit-msg git hook. Everything is additive (merges into existing files, never overwrites). Detects and migrates stale hook formats from older installs.

### `apps/desktop` — the UI

A Tauri app (Rust shell + React/TypeScript frontend). All mutations route through `packages/core` via the host sidecar — there is no private store, so closing the app loses nothing.

- **Views**: Welcome, Board (drag-and-drop columns), List, TicketDetail (schema-generated forms), Documents (navigable tree), Graph (wiki-link visualization), Settings (schema editor).
- **Components**: `FieldInput` renders the correct input per field type at runtime (enum → select, date → date picker, reference → entity picker, list → tag input). `InitWizard` scaffolds new projects. `LiveRing` shows agent presence.
- **Editor**: Lexical-based block editor with Markdown round-trip conversion. Custom nodes for wiki-links, verbatim blocks, and Mermaid diagrams. Slash menu for block insertion.
- **Design system**: Tokens in `tokens.css`. Dark default, light via token flip. Three commitments: no borders (separation via tone), one electric signal (cyan, reserved for agent activity/selection/focus), weaving-and-punchcards as interaction language. Geist Sans throughout, Geist Mono only for code.

---

## Key Techniques

**Schema-driven UI at runtime**: Ticket fields are data in `schema.yaml`, not code. All four consumers — the validator, the indexer, the MCP server, and the app's generated forms — read field definitions at runtime. Only five core fields are locked (`id`, `type`, `status`, `created`, `updated`). Users add custom fields by editing YAML.

**Deterministic indexing**: `buildIndex()` sorts every list, fixes every object key order, and never emits timestamps or random values. Indexing unchanged input twice produces byte-identical output. This makes `index.json` safe to regenerate from scratch at any time and guarantees git doesn't see noise from re-indexing.

**Atomic ID assignment via mkdir lock**: Uses `mkdirSync` on a lock directory — mkdir is the universal atomic operation. Retries every 25ms up to 5 seconds. Missing counter files are rebuilt by scanning all existing entity filenames across tickets, sessions, and documentation.

**Tolerant reads, non-destructive writes (ADR-0011)**: Every Zod schema uses `.passthrough()`. Unknown frontmatter keys and schema constructs are warnings, never errors. When the schema editor saves, it matches new nodes to old ones (by name or through rename maps) and carries unknown keys forward. A newer-minor construct the current tooling doesn't understand survives a save untouched.

**Round-trip fidelity**: `parseFrontmatter()` preserves original line endings by slicing the body from the original bytes rather than re-joining. `updateTicket()` parses the existing YAML frontmatter with `parseDocument()`, sets only changed keys, and re-serializes — preserving whitespace, comments, and key order in the frontmatter. The block editor must produce byte-identical output when no edits are made.

**Agent-native orientation**: The digest system (`digest.ts`) produces a ~1,500-token plain text summary: what's in progress, what's ready, recent session records with their open questions, and validation warnings. This is injected at session start via a Claude Code hook, so every new agent session starts oriented without the agent needing to discover context on its own.

**Presence heartbeat with automatic GC**: Each agent session writes a per-session marker (`state/presence/<session-id>.json`) with `started_at` and `beat_at`. `beatPresence()` updates the heartbeat on every tool call (the hot path, deliberately skips `loadProject`). `writePresence()` runs garbage collection: entries whose last beat exceeds `presence_timeout_minutes` (default 15) are removed. The app reads these markers to show live agent activity.

**Three-tier sidecar isolation**: The app never imports `packages/core` directly. It spawns the host sidecar per operation (JSON request → JSON response → exit). The MCP server runs as a persistent stdio process. The helper runs as short-lived hook commands. This isolates the Rust shell from Node.js and keeps the app's runtime state in the filesystem, not in memory.

**Claude Code hook chain**: Seven hook commands working together — SessionStart (digest injection), Stop (session-check: ensures record written and ticket moved), PreToolUse (guard: blocks direct edits to `.lovelace/tickets/`), UserPromptSubmit (presence start), PostToolUse (heartbeat + per-session active ticket tracking), SessionEnd (presence clear). Plus slash commands `/ticket` (load, orient, start) and `/done` (verify acceptance criteria, move, record).

---

## Design Decisions

**Files ARE the database, not a serialization of one**: The entire project state lives in `.lovelace/` as plain Markdown. There's no import/export — cloning the repo gives you the full project. Hand-editing files is a supported path; malformed input produces clear validation errors with file and line, never silent repair or crash.

**Optimised for agent context, not human browsing**: Documents carry `summary` fields so consumers (indexes, the MCP `read_document` tool) never need to parse bodies. The digest is a compact orientation, not a visual dashboard. Session records are structured for the next agent session, not for management reporting.

**Schema versioning as a contract**: Spec version is semver. Tooling refuses major versions it doesn't understand (clear error: "update Lovelace" or "migrate the project"). Minor versions are additive and tolerated — unknown constructs produce warnings and survive writes. The declared version is a floor, never auto-bumped; only migrations rewrite it.

**Flat ticket directory**: Tickets never move directories when status changes. This keeps git diffs clean and hand navigation simple. Filtered views come from the index, not from the filesystem layout.

**No transition automations**: Removed in spec 3.0. Any ticket may move to any status. Lovelace explicitly describes itself as "an orchestrator, not a CI system" — no retries, queues, or scheduling. This eliminates a class of complexity that the v1 scope (solo developer) doesn't need.

**v1 scope is deliberately narrow**: Solo developer, one repo per project, current branch only. No multiplayer, sync, branch switching, web views, or multi-repo. The trade-off is depth over breadth — the agent integration is unusually polished for a v1.

**Self-dogfooding from day one**: The Lovelace repo manages itself with Lovelace. The 139 tickets and 144 session records behind the app live in `.lovelace/` at the repo root. This forces the team to experience every rough edge directly.

**Constraint-driven design language**: The app enforces a strict visual system — no borders (separation via tonal steps), one electric signal color (cyan), Geist Sans with exactly four sizes, cards limited to title + one meta row, motion capped at 450ms. This is an explicit reaction to the visual and cognitive bloat of Jira and Linear.

---

## Comparison Notes

Unlike **[[Bram]]** which is a Tauri desktop shell for agentic development with hash-verified worklists, Lovelace is a full project management system — the worklist IS the database, not a checked artifact. Bram enforces process through verification; Lovelace encodes process in schema-driven definitions.

Unlike **Jira** and **Linear** which are cloud-hosted SaaS with accounts, Lovelace is local-first with no server component. The entire project state is files in the repo, readable without the app. This makes it more akin to **Obsidian** (local-first, file-based, Markdown) applied to project management rather than personal knowledge management.

Unlike general **MCP servers** that expose tools for arbitrary services, Lovelace's MCP server is purpose-built for project management and deeply wired into Claude Code via hooks, slash commands, and git integration. The deliberate eight-tool cap is a scope constraint, not a limitation.

Unlike [[Agent Coding Workflow]] strategies that focus on code generation, Lovelace focuses on project continuity — session records, open questions, and digest-based orientation ensure each new agent session starts meaningfully smarter. The product thesis is that putting project management where agents already live (the filesystem and Git) gives them native context without additional integration layers.

Unlike [[Teaching the Agent Our Craft]] which encodes process through skills and stories, Lovelace encodes process through schema definitions (custom fields, statuses with agent roles) that are data, not imperative instructions. The agent discovers process by reading the schema, not by following a skill.

---

*Sources: [[raw/lovelace]]*
*Last updated: 2026-07-25*
*Tags: #tool #project #agents #claude-code #mcp #local-first*
