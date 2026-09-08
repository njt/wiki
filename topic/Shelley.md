# Shelley

A mobile-first, web-based, multi-conversation, multi-modal, multi-model, single-user coding agent built by Bold Software for exe.dev. Shelley is the concrete realization of the "bring the agent to the compute" thesis — a Go backend with SQLite storage and a Vue 3 + PrimeVue frontend compiled into one binary that serves a browser UI over SSE. It has no auth and no sandboxing: those are deliberately left to the host (exe.dev provides the VM). It is, architecturally, the exe.dev GUTS stack made manifest — and it ports Mario Zechner's compaction prompt verbatim from pi-mono.

---

## Architecture

Shelley is a **five-layer Go monolith with a thin TypeScript UI**:

- **`cmd/shelley`** — a thin shim that starts the server.
- **`server/`** — the HTTP API, an in-memory map of active conversations, the SSE fan-out, compaction, and subagent orchestration.
- **`loop/`** — the agentic loop: iterative `processLLMRequest` calls, tool execution, retries, and timeouts.
- **`claudetool/`** — the tool layer (a 10-tool `ToolRegistry`), the naming an artifact of its Claude Code lineage.
- **`llm/`** — a unified `Service` interface over three providers (OpenAI, Anthropic, Gemini) plus a deterministic test fixture.

Persistence is **SQLite via sqlc codegen** with 37 migrations; the `Conversations → Messages` model records messages from four origins — user, model, tool, and harness — all in one table. The UI is **embedded into the binary** with `embedfs`, so a single executable serves both the API and the frontend.

The realtime story is a **unified `/api/stream2` SSE endpoint** carrying per-conversation events plus **RFC-6902 JSON Patch diffs** of the conversation list, reconciled by snapshot-and-hash. The last 100 patch events are retained for reconnect replay, and a custom `SubPub[K]` pub-sub layer fans events out to subscribers.

## Key Techniques

### The agentic loop as an iterative process, not recursion

`loop/loop.go` runs `processLLMRequest` in a flat iteration rather than recursive re-entry, avoiding the O(n²) context growth that naive recursive loops produce. Each round applies prompt caching (the `Cache` flag on the last tool result and last user message), and runs **sibling tool calls concurrently** behind a start barrier. A 15-minute `maxTurnDuration` backstop sits alongside idle/stall timeouts and a `toolCancelGrace` of 1 second.

### Error taxonomy driving retry policy

Errors are typed (`ErrorType`: truncation / llm_request / refusal) with an `ExcludedFromContext` flag and a two-tier retry distinction — `isRetryableError` (transport, auto-retried) vs. `IsRetryableLLMError` (the user-facing retry button). Refusals are non-retryable and kept out of the context. Missing tool results get **synthetic results** (`insertMissingToolResults`) with orphan filtering, so the model is never left waiting on a tool call that vanished.

### Subagents via completion splicing and watermarks

Rather than a separate runtime, subagents are synthesized by splicing `tool_use`/`tool_result` messages into the parent conversation, coordinated with **supersession watermarks** (`handledSeq`, `claimNotified`, `dropStaleParentNotification`) so a stale parent notification can't clobber a newer child result. `wait=true/false` plus 500ms polling and progress summaries give both blocking and fire-and-forget modes; `cancelSubagentTree` tears down the whole tree.

### Compaction ported from pi-mono

`server/distill_pi.go` ports badlogic/pi-mono's distillation algorithm nearly verbatim: a **structured summary prompt** (Goal / Constraints / Progress / Key Decisions / Next Steps / Critical Context), `keepRecentTokens=20000` of verbatim recent context, `reserveTokens=16384` for the summary, a `chars/4` token heuristic, and a cut point that never lands on a `tool_result`. On failure it rolls back; on refusal it falls back from Fable to Opus; the whole swap is a single-transaction write.

### The shell as the primary tool

`claudetool/shell.go` models shell commands with a **`yield_time_seconds`** parameter (default 30s, max 10m) so the model budgets how long it's willing to wait. Background processes get PID/PGID/log tracking with `Setpgid` process-group kills, a live tail progress loop, a `GIT_SEQUENCE_EDITOR` guard, and an automatic Co-authored-by trailer.

### Multi-model behind one interface

The `llm.Service` interface abstracts OpenAI, Anthropic, and Gemini behind `Message`/`Content`/`Tool`/`ThinkingLevel` types, with a `ThinkingBudgetTokens` mapping and a **custom Lark grammar** for OpenAI tool schemas. Hooks are executable scripts in `~/.config/shelley/hooks/` (system-prompt, new-conversation, chat-message, end-of-turn); skills are Markdown `SKILL.md` files; notification channels cover discord, email, and ntfy.

## Design Decisions

**Bring the agent to the compute.** Single-user by design: exe.dev hands Shelley a VM, so there's no tenant isolation, no sandboxing, no auth in the codebase. That's a real simplification of a class of problems every multi-tenant agent (Claude Code's permission system, Codex's sandbox, omp's pi-iso) must solve — Shelley simply declares them out of scope.

**Web-first, not terminal-first.** The README's wry justification ("terminal scrollback is punishment for shoplifting in some countries") is a genuine architectural bet: a browser UI on a phone means the agent is reachable any time, and the SSE stream is the contract, not the terminal. This is the inverse pole of [[Grok Build]]'s TUI-first investment.

**SQLite + sqlc over an ORM.** 37 migrations, generated type-safe query code, one file on disk — the same "boring, legible, agent-friendly" substrate [[The GUS Stack — Go, Unix, SQLite]] prescribes, and Shelley is literally the exe.dev GUTS (Go, Unix, TypeScript, SQLite) stack realized.

**A curated error/retry semantics.** The two-tier retry distinction (transport auto-retry vs. user retry button) and refusal-out-of-context handling are where much of the agent's reliability actually lives — quieter than tools or models, but decisive for long-running turns.

## Comparison Notes

- **vs. [[Pi Coding Agent]]**: Shelley ports pi-mono's compaction prompt nearly verbatim, but otherwise diverges hard — Pi is a minimal terminal harness in TypeScript, Shelley is a web-delivered Go monolith. The shared compaction is a direct lineage link.
- **vs. [[The GUS Stack — Go, Unix, SQLite]]**: Shelley is the exe.dev GUTS stack made concrete — Go backend, SQLite via sqlc, TypeScript/Vue frontend — exactly the "shared, boring, agent-legible foundation" the article argues for.
- **vs. [[Grok Build]]**: Grok Build is a ~1M-line pure-Rust TUI monolith with a three-strategy compaction engine; Shelley is a web-first Go monolith with a single (pi-ported) compaction strategy. Both single-binary, but opposite surface bets.
- **vs. [[Components of a Coding Agent]]**: Shelley is a clean case study for the "harness matters more than the model" thesis — its loop, timeouts, retry taxonomy, and subagent watermarks are harness engineering, largely model-agnostic behind the `llm.Service` interface.

---

*Sources: [[raw/shelley]], [[summary/shelley]]*
*Last updated: 2026-09-08*
*Tags: #tool #project #coding-agents #go #sqlite #vue #web #sse #multi-model*
