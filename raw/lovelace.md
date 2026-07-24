---
url: https://github.com/lovelace-co/lovelace
title: Lovelace
author: ADACA PROJECTS PTY LTD
date_fetched: 2026-07-25
date_published: 2026-06-10
---

# Lovelace

Local-first project management for software teams that build with AI coding agents. A Tauri desktop app (macOS, Linux, Windows) where tickets, documentation, architecture decisions and agent session history live as plain Markdown files with YAML frontmatter in a `.lovelace/` directory inside the repository. No server, no database, no account, no cloud. AGPL-3.0.

## Architecture

Monorepo (pnpm workspaces, Node 22, TypeScript strict):

### packages/core
Pure logic — parsing, schemas, validation, indexing, digest, mutations. Uses `yaml` (with LineCounter for error positions), Zod for runtime schema validation. Key modules:
- `frontmatter.ts`: Parses YAML frontmatter from Markdown, preserving line endings (LF/CRLF/CR). Returns key-to-line mapping for precise error reporting.
- `config.ts`: Loads and validates manifest.yaml, schema.yaml, actors.yaml via Zod schemas with `.passthrough()` everywhere (tolerant reads, non-destructive writes per ADR-0011).
- `validate.ts`: Full project validation — ID format, filename matching, type/status existence, field validation against schema definitions, link integrity, review staleness, session completeness.
- `fields.ts`: Runtime field validation driven by schema.yaml definitions. `validateTicketFields()` checks types, required fields, enum values, reference format. `collectReferences()` gathers values for link integrity checking. `applyDefaults()` fills in declared field defaults.
- `index-gen.ts`: Builds deterministic `index.json` (sorted keys, no timestamps). Also generates `BOARD.md` (committed Markdown board grouped by status).
- `links.ts`: Wiki-link graph extraction via regex `[[id]]` scan. Excludes `[[file:...]]` and `[[@actor]]` references. Deduplicated, sorted, deterministic.
- `mutate.ts`: All state mutations — create/update/delete ticket, log session, add comment. ID assignment via `ids.ts` with exclusive lock directories (mkdir is atomic). Schema editor with rename cascading. Presence management with heartbeat and GC. Per-session active ticket markers.
- `digest.ts`: Compact orientation summary (~1500 tokens) — in-progress tickets, ready work, recent sessions with open questions, validation warnings/errors.
- `ids.ts`: Sequential ID assignment with mkdir-based exclusive lock and scan-based recovery of highest existing number.
- `watch.ts`: Debounced file watcher that re-validates and re-indexes on change.

### packages/mcp
Three agent-facing processes, shipped as Bun-compiled sidecar binaries:
- `server.ts` (lovelace-mcp): MCP server on stdio transport (official TypeScript MCP SDK). Eight tools: create_ticket, update_ticket, describe_schema, query_tickets, read_document, log_session, set_active_ticket, search. Tool descriptions are generated at registration from schema definitions — including per-field value hints with enum values resolved.
- `helper.ts` (lovelace-agent): Headless helper invoked by Claude Code hooks — digest, session-check, guard (blocks direct ticket edits), presence-start/beat/clear, track-active. Reads JSON payload from stdin (hook context including session_id). All presence ops fail open (never block a turn).
- `host.ts` (lovelace-host): Stateless JSON bridge between app's Rust shell and core. One request in, one response out, process exits. Handles ~30 operations including ping, detect, init, snapshot, migrate, create/update/delete ticket, schema/manifest/actor edits, Claude/OpenCode integration install/detect, file ops, git commit lookup.
- `claude.ts`: Generates and installs Claude Code assets — CLAUDE.md marker section, `.mcp.json`, hooks in `.claude/settings.json`, slash commands (`/ticket`, `/done`), opt-in prepare-commit-msg git hook. Everything is additive (merge, never overwrite). Detects and migrates stale hook formats.
- `assets.ts`: Shared asset utilities for Claude/OpenCode integration.

### apps/desktop
Tauri app (Rust shell + React/TypeScript frontend):
- Views: Welcome, Board, List, TicketDetail, Documents, Graph, Settings
- Components: InitWizard, FieldInput (schema-driven), SearchPalette, ContextMenu, BulkActions, Presence indicators (LiveRing)
- Editor: Lexical-based block editor with Markdown round-trip conversion, WikiLinkNode, VerbatimNode, MermaidNode, SlashMenu
- State: Application store, theme support (design tokens in tokens.css)
- Lib: host IPC, datetime (OS locale), links, presence, force graph layout, formatting resolvers

### examples/demo-project
Canonical fixture exercising every entity type; tests run against it and deliberately corrupted variants.

## Key Techniques

**Schema-driven UI at runtime**: Ticket fields are data in `schema.yaml`, not code. The validator (`fields.ts`), indexer (`index-gen.ts`), MCP server (`server.ts` generates tool descriptions with field hints at registration), and the app's forms (`FieldInput.tsx`) all read field definitions at runtime. Only five fields are locked: `id`, `type`, `status`, `created`, `updated`. This means users can add custom fields without code changes.

**Deterministic indexing**: `buildIndex()` sorts every list, fixes every key order, and never emits timestamps. Running the indexer twice on unchanged input produces byte-identical output. This makes index.json safe to regenerate and never produces noise in git.

**Atomic ID assignment with mkdir lock**: `ids.ts` uses `mkdirSync` on a lock directory — mkdir is atomic on all platforms. Retries with 25ms sleep up to 5s timeout. Missing counters are rebuilt by scanning existing entity IDs across tickets, sessions, and documentation directories.

**Tolerant reads, non-destructive writes (ADR-0011)**: Every Zod schema uses `.passthrough()` so unknown keys survive parsing. `validateSchema()` warns on unknown keys but never errors. `writeSchema()` carries forward unknown constructs from matched old schema nodes, so a newer-minor construct that this version doesn't recognise isn't silently destroyed. The manifest uses `MANIFEST_KNOWN_KEYS` (derived from Zod shape) to identify unrecognised keys.

**Round-trip fidelity preservation**: `parseFrontmatter()` slices the body from the original text (not re-joined from split lines) to preserve CRLF. `updateTicket()` parses the existing YAML with `parseDocument()`, sets only changed keys, and re-serializes — so whitespace and comments in the YAML survive untouched. The app's block editor requires byte-identical output when no edits are made.

**Presence system with heartbeat GC**: Each agent session writes a per-session marker in `state/presence/<session-id>.json` with `started_at` and `beat_at`. `beatPresence()` is the lightweight path (called on every tool use, skips `loadProject` — only parses manifest.yaml to resolve state/). `writePresence()` runs GC: entries whose last beat exceed `presence_timeout_minutes` (default 15) are removed. Legacy singleton `presence.json` is cleaned up on write.

**Agent-native digest injection**: `buildDigest()` produces a ~1500-token plain text summary: in-progress tickets (statuses with `agent: in_progress`), ready work (`agent: ready`), last 3 session records with open questions, and validation warnings. Injected by a SessionStart hook.

**Three-tier sidecar architecture**: The desktop app never calls core directly. It spawns the host sidecar per operation (one JSON request → one JSON response → process exits). The MCP server runs as a persistent stdio process. The helper runs as short-lived hook commands. This isolates the Rust shell from Node.js and keeps the app's state in the filesystem, not memory.

**Claude Code deep integration**: Self-installing hooks chain — SessionStart (digest), Stop (session-check: ensures record written, ticket moved), PreToolUse (guard: blocks direct ticket edits), UserPromptSubmit (presence start), PostToolUse (presence heartbeat + active ticket tracking), SessionEnd (presence clear). Slash commands `/ticket` and `/done` with multi-step agent instructions. Git hook injects active ticket ID into commit messages.

## Design Decisions

**Files as the database, not a serialization of one**: The `.lovelace/` directory IS the database. Every entity is one file. Hand-editing is a supported path. Malformed input produces validation errors with file and line, never a crash. There is no import/export — you clone the repo and you have the project state.

**Optimised for agent context, not human browsing**: The digest system, session records, wiki-link graph, and MCP tools are designed for agents to consume. Documents carry `summary` fields so consumers never need to parse bodies. The digest is a compact orientation, not a dashboard.

**Schema versioning as a contract, not a version number**: The spec is semver-versioned. Tooling refuses major versions it doesn't understand (clear error: update the app or migrate the project). Minor versions are additive and tolerated (unknown keys preserved through writes as warnings). The declared version is a floor, never auto-bumped.

**Flat directory structure for tickets**: Tickets never move directories when status changes. This keeps git diffs clean and makes hand navigation simple. The index provides filtered views; the filesystem is just storage.

**No transition automations**: Removed entirely in spec 3.0. A ticket may move to any status. Lovelace explicitly positions itself as "an orchestrator, not a CI system" — no retries, queues, or scheduling. Transition automations were deemed overengineering for the v1 scope.

**v1 is solo developer only**: No multiplayer, sync, branch switching, web views, or multi-repo. The product thesis is that coding agents already live in the filesystem and Git — putting project management there too gives them native context.

**Constraint-driven design language**: The app enforces a strict visual system — no borders (separation via tone), one electric signal color (cyan), Geist Sans with exactly 4 sizes, cards limited to one title row + one meta row, motion capped at 450ms. This is a reaction to tool bloat in Jira/Linear.

## Comparison Notes

Unlike **Bram** which is a Tauri desktop shell for AI-assisted development with hash-verified worklists and PreToolUse hooks, Lovelace is a full project management system — the worklist IS the database, not a checked artifact. Bram's worklist lifecycle enforces process; Lovelace leaves process to the project's schema definition.

Unlike **Jira** and **Linear** which are cloud-hosted, account-based SaaS products, Lovelace is local-first with no server component. The entire project state is files in the repo, readable without the app. This makes it akin to **Obsidian** (local-first, file-based, Markdown) applied to project management rather than personal knowledge management.

Unlike general **MCP servers** which expose tools for arbitrary services, Lovelace's MCP server is purpose-built for project management and deeply integrated with Claude Code via hooks, slash commands, and git integration. The eight-tool limit is a deliberate scope constraint.

Unlike **agent coding workflows** that focus on code generation, Lovelace focuses on project continuity — session records, open questions, and digest-based orientation ensure each new agent session starts meaningfully smarter than the last.

---
*Source: https://github.com/lovelace-co/lovelace*
*Last updated: 2026-07-25*
