# Local Review

A local, single-user git review tool that gives you GitHub-style review ergonomics for branches you haven't pushed — or don't want to push — entirely on your machine. Point it at a folder of repos, pick a branch, comment on any line or range, reply in threads, resolve what's done, and export the review as clean markdown for a coding agent to act on. Ships as a single Go binary with the React frontend embedded; no runtime dependencies beyond `git`.

---

## Architecture

A **single Go binary** (`main.go`, 143 lines) serves a JSON API over `net/http` (Go 1.22+ method+path routing) and mounts an embedded React/TypeScript frontend via `//go:embed all:web/dist`. The server listens on `127.0.0.1:7777` and opens the browser automatically.

Four internal packages handle the core logic:

- **`internal/git/git.go`** (451 lines) — wraps the real `git` binary via `os/exec`. Parses `git diff` output with a custom parser that correctly handles edge cases (`---`/`+++` appearing in content like SQL comments or added `++` lines, rename chains, binary detection, `\ No newline` escape lines). Also handles working-tree diffs by supplementing `git diff <base>` with `git ls-files --others --exclude-standard -z` for untracked files.

- **`internal/store/store.go`** (629 lines) — SQLite via `modernc.org/sqlite` (pure Go, no cgo), WAL mode enabled. Uses `SetMaxOpenConns(1)` — a single connection ensures `PRAGMA foreign_keys=ON` is authoritative and transactions are atomic. Schema: `reviews`, `comments`, `replies`, `reviewed_files` with FK cascades. Idempotent migration via `ensureColumn()` that checks `PRAGMA table_info` before `ALTER TABLE ADD COLUMN`. Reviews resume by `(repo_path, base_ref, head_ref)` regardless of `status`.

- **`internal/api/api.go`** (751 lines) — HTTP handlers for repo listing, branch enumeration, diff serving, file/blob serving, CRUD for reviews/comments/replies, reviewed-file toggling, export, and SSE event streaming. All mutation handlers publish to the SSE hub. Comment creation records `commit_sha` (best-effort, resolved at add time) and `worktree` flag for correct anchor-side comparison.

- **`internal/api/annotate.go`** (192 lines) + **`reviewed.go`** (63 lines) — derive comment staleness and reviewed-file validity on every read. Comment anchoring uses two tiers: precise git line tracking via `git diff <commit_sha> head -- path` with `MapOldLine` (primary), falling back to snippet matching. File reviewed status uses SHA-256 content fingerprints — files that change after being marked reviewed automatically revert to unread.

- **`internal/api/events.go`** (57 lines) — in-memory SSE hub with per-review subscriber maps, size-1 buffered channels for coalescing, non-blocking sends, and auto-prune on last unsubscribe.

- **`internal/export/export.go`** (219 lines) — canonical markdown formatter. Groups comments by file, excludes resolved threads, computes backtick fences one longer than the longest backtick run in any snippet (CommonMark-safe). Optionally appends agent reply instructions with a concrete `curl` example.

The **React frontend** (`web/src/`, ~1,100 lines) is a three-column resizable layout: hierarchical file tree (left) → per-file diff view (center) → comment overview panel (right). Uses Shiki for syntax highlighting (~235 languages, JS regex engine to avoid WASM load failures), IntersectionObserver for viewport-mounting of lazy files, and `markdown-it` for export preview rendering.

## Key Techniques

### Drift-resistant comment anchoring

Comments survive branch rebasing and further commits through a two-tier system:

1. **Precise git line tracking**: `annotateByDiff()` runs `git diff <commit_sha> head -- path` and maps the original line range through the diff hunks with `MapOldLine()`. Every line surviving contiguously → `current` or `moved` (relocated range provided); any line deleted or modified → `outdated`. This is preferred because it distinguishes genuine moves from coincidental reappearances of the same lines — something snippet matching structurally can't do.

2. **Snippet matching fallback**: `annotateComment()` compares the captured code snippet against the current file (working tree or head, depending on the `worktree` flag recorded at comment time). Unambiguous re-match → `moved` (with new line range); gone or ambiguous → `outdated`.

Diffs and file reads are cached per `(commit_sha, path)` or per `path` within a single review read, so the O(N×M) cost is amortized.

### Derived state, never persisted

Comment anchor status and reviewed-file validity are recomputed on every read, never trusted from stored flags. The `anchorStatus`/`currentStartLine`/`currentEndLine` fields on the `Comment` struct have `omitempty` JSON tags; the store writes and reads zero values for them. `annotateReview()` fills them in the API layer before serializing to JSON. This means force-pushes, rebases, and interleaved commits are always reflected correctly — no stale "reviewed" checkmarks after new work.

### Agent handoff as API, not just export

The tool supports two agent handoff modes:

- **Copy agent instructions**: copies a prompt telling a coding agent to `POST /api/reviews/{id}/export` (reading the `markdown` field) and reply to individual comments via `POST /api/comments/{id}/replies`. The agent fetches fresh state on each iteration — no re-paste needed when the human adds or changes comments.

- **Static export**: preview, then copy or download the rendered markdown. Optionally includes `curl` examples so a paste-only agent can still reply.

Either way, agent replies appear live in the UI via SSE, so the human can read them, resolve what's addressed, and hand off what's left.

### Live multi-tab sync via SSE

The SSE hub recognizes that a dropped ping is harmless when every ping triggers a full refetch — at most one refresh is owed per client. Size-1 buffered channels with non-blocking sends mean a stalled tab never blocks a mutation handler. Keepalive comments (every 25s) turn half-open connections into write errors that trigger unsubscribe. The frontend supplements with a focus/visibility refetch for the reconnect gap.

### Backtick-safe markdown export

`fenceFor()` scans each captured snippet for the longest consecutive run of backticks, then emits a fence one longer (minimum 3). This prevents a reviewed markdown file containing triple-backtick code fences from prematurely closing the export's code block — a concrete edge case handled by a general algorithm.

### Responsiveness for large change-sets

Four independent mechanisms keep the UI responsive on large diffs:
- **LazyFile viewport-mounting** via `IntersectionObserver` — off-screen files never fetch, tokenize, or render
- **Auto-collapse** for files with >500 changed lines
- **Highlight skip** for files >2000 lines
- **DOM-level panel resize** during drag (writes `grid-template-columns` directly, no React re-render per `mousemove`)

## Design Decisions

**Shells out to git binary rather than using go-git.** Correctness on real repos (renames, `.gitattributes`, submodules) beats theoretical portability. The tool's entire purpose is reading git state; using the real git is the obvious call.

**Single SQLite connection (`SetMaxOpenConns(1)`).** `foreign_keys` pragma is per-connection in SQLite — a pooled DB would have some connections miss it and silently skip `ON DELETE CASCADE`. Single connection makes the pragma authoritative and gives transactions full atomicity. DB access is low-frequency (git operations don't hit SQLite), so serialization costs nothing.

**Backend as canonical source of truth.** The frontend is a cache that refetches on SSE pings. This means derived state (comment staleness, reviewed-file validity) is computed server-side on every read — no conflict resolution logic in the client, no risk of stale client-side caches. The trade-off is network round trips, but on localhost that's negligible.

**Recompute staleness on read rather than persist at write time.** Updating all comments on every push would be expensive and fragile. Computing on read means a freshly pulled branch always shows correct anchor status. The cost is O(comments × diffs) per read, which is fine for human-scale reviews (tens to hundreds of comments).

**Two-level threading (comment + replies) rather than arbitrary nesting.** Simple, predictable, no edge cases. FK cascades ensure replies never orphan. GitHub-style deep threading is complex to render and rarely needed for agent-bound review feedback.

## Comparison Notes

Unlike **[[OpenCodeReview]]**, which uses AI to *generate* review comments, local-review is for *humans to write* comments that an AI coding agent then acts on. The direction is inverted.

Unlike **[[Bram]]**, which layers a two-gate approval workflow over coding agent sessions, local-review is purely a review artifact generator — it produces markdown, which can feed into any agent but doesn't execute or approve anything itself.

Unlike **[[DeltaDB]]**, which traces every line to the conversation that produced it, local-review only captures the *review moment* — a snapshot of human judgment at a point in time.

Unlike **[[Metis — ARM AI Security Code Review]]** and **[[brooks-lint]]**, which use deterministic analysis or prompt engineering to find issues, local-review makes no attempt to understand code semantics. It operates at the diff and line level, leaving all understanding to the human reviewer and the receiving coding agent.

The tech stack echoes **[[Memento]]** (Go + SQLite + SSE runtime), though for a different domain — Memento is a knowledge layer; local-review is a review layer.

Placed in the [[Agent Coding Workflow]] pipeline, local-review sits at the **human review** step between code generation and agent iteration — the artifact that closes the loop between "AI wrote this" and "human verified this, now fix these things."

[[Hunk]] occupies the complementary role in the pipeline: where Local Review captures human feedback for agents to consume, Hunk renders agent annotations for humans to read. Together they form the two halves of a bidirectional review loop — agent→human (Hunk) and human→agent (Local Review) — with [[Subspace]] providing an alternative human→agent path for structured terminal-based feedback.

---
*Sources: [[raw/local-review]]*
*Tags: #tool #project #developer-tools #code-review #git*
*Last updated: 2026-07-11*
