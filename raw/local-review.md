---
url: https://github.com/rosenbjerg/local-review
title: local-review
author: rosenbjerg (Malthe Rosenbjerg)
date_fetched: 2026-07-11
date_published: 2026-06
---

# local-review

A local, single-user git review tool: review a branch's diff, leave line/range comments, mark files reviewed, and export the review as markdown for a coding agent. Go backend + React frontend, shipped as **one binary** (the built frontend is embedded via `go:embed`).

## Architecture

### Stack
- **Go 1.26.3** backend with **React/TypeScript** frontend embedded via `embed.FS`
- **SQLite** via `modernc.org/sqlite` (pure Go, no cgo), WAL mode enabled
- **Shells out to `git`** via `os/exec` — correct on renames, submodules, `.gitattributes` diff filters; less code than go-git
- Single binary, no runtime dependencies beyond `git`

### Server (`main.go`, 143 lines)
- Flags: `-root` (folder of repos), `-port` (7777), `-data-dir` (`~/.local-review`), `-retention-days` (30), `-no-open`
- DB path resolves `~`, creates data directory if needed
- Prunes draft reviews older than N days on startup
- Mounts static handler with SPA fallback for client-side routing
- Opens browser after 300ms delay

### Git service (`internal/git/git.go`, 451 lines)
- `Repo` handle wrapping `os/exec` calls to git binary
- `ListBranches()`: parses `git branch --format=...`, flags current and main branches, sorts pinned branches (main/master/develop/dev/staging) to top
- `MainBranch()`: prefers local main/master, falls back to origin/HEAD → origin/main → origin/master, returns "" if nothing resolves
- `MergeBase()`, `ResolveSHA()`: thin git wrappers
- `FileContent(ref, path)`: `git show ref:path`
- `WorktreeFile(path)`: reads on-disk file with traversal guard (rejects `.git` and `..` paths)
- **Custom diff parser** (`parseDiff`): handles `---`/`+++` appearing in content (e.g. SQL comments), rename lines, binary detection, NUL-byte detection. Outputs `FileDiff` structs with `Hunks` containing `DiffLine` entries.
- `DiffWorktree(base)`: `git diff <base>` + `git ls-files --others --exclude-standard -z` for untracked files, synthesizes `FileDiff`s from on-disk content
- `MapOldLine(hunks, old)`: maps old-side line through diff hunks to new-side line, returns `alive=false` for deleted/modified lines
- `diffPrefixArgs`: forces canonical `a/` and `b/` path prefixes regardless of user config (mnemonicPrefix, noprefix, etc.)

### Store (`internal/store/store.go`, 629 lines)
- `Store` wrapping `database/sql.DB` with `SetMaxOpenConns(1)` — single connection ensures `PRAGMA foreign_keys=ON` is authoritative and transactions are atomic
- **Schema**: `reviews`, `comments`, `replies`, `reviewed_files` tables with FK cascades
- **Migration**: `CREATE TABLE IF NOT EXISTS` + `ensureColumn()` which checks `PRAGMA table_info` before `ALTER TABLE ADD COLUMN` — idempotent across upgrades
- `CreateOrGetReview`: transaction-wrapped check-then-insert, matched by `(repo_path, base_ref, head_ref)` regardless of status
- `AddComment`: stores `commit_sha` (best-effort resolve at add time) and `worktree` flag
- `SetFileReviewed`: upserts with content fingerprint (SHA-256) and worktree side flag
- `SetCommentResolved`: toggles `resolved` bit on root comment
- `PruneDrafts`: deletes reviews with `status='draft'` older than retention window
- Replies FK-cascade from comments, comments cascade from reviews

### API handlers (`internal/api/api.go`, 751 lines)
- Go 1.22+ method+path routing (`GET /api/repos`, `POST /api/reviews/{id}/comments`, etc.)
- `repoFor()`: validates repo name is a single path segment under root (guards against traversal)
- `validRef()`: rejects empty and `-`-prefixed refs (prevents git flag injection)
- `handleDiff`: resolves base to merge-base at query time, supports `?uncommitted=true` for working tree diff
- `handleFile`: reads file at ref or working tree, with working-tree fallback on failure
- `handleBlob`: raw bytes with `Content-Type` for image rendering (png/jpg/gif/webp/bmp/ico/avif/svg + stdlib fallback)
- `handleCreateReview`: resolves base to main branch name if empty, stores `head_sha`, annotates review
- `handleExport`: renders markdown via `export.Render`, marks review `exported`, returns `{markdown, filename}`
- `handleSetReviewed`: captures SHA-256 fingerprint of file content, stores with worktree side flag
- `handleAddComment`: resolves commit_sha at add time, annotates new comment, publishes SSE ping
- Default author: `"agent"` when omitted (coding agent via API), `"reviewer"` when browser creates

### SSE events (`internal/api/events.go`, 57 lines)
- In-memory hub keyed by review ID
- `subscribe()`: creates size-1 buffered channel (coalesces bursts)
- `publish()`: non-blocking send, so a stalled tab never blocks a mutation handler
- `unsubscribe()`: auto-prunes review entry when last subscriber disconnects
- `handleEvents`: 25s keepalive comments, exits on context done or write error

### Comment anchoring (`internal/api/annotate.go`, 192 lines)
- `annotateReview()`: fills derived fields on every review read
- **Primary: precise line tracking via git** (`annotateByDiff`): runs `git diff <commit_sha> head -- path`, maps original range through hunks with `git.MapOldLine()`. Every line surviving contiguously → `current` or `moved`; any deleted/modified → `outdated`. Results cached per (commit_sha, path) per review read.
- **Fallback: snippet matching** (`annotateComment`): reads current file lines (head or working tree depending on `worktree` flag), matches captured snippet. Match at stored range → `current`; unique match elsewhere → `moved`; gone/ambiguous → `outdated`. Uses lazy file reader with per-path cache.
- Snippet matching uses `strings.Split` for line comparison, `findMatches` returns all starts, unambiguous hits only

### Reviewed file tracking (`internal/api/reviewed.go`, 63 lines)
- `annotateReviewedFiles()`: re-verifies every reviewed-file mark by hashing current content
- `fileContentHash()`: SHA-256 of current file content (head or working tree), empty string on failure
- Empty fingerprint (older rows) stays reviewed; changed fingerprint drops the mark
- Mirror of comment anchoring pattern: derived, never trusted from flag alone

### Markdown export (`internal/export/export.go`, 219 lines)
- `Render()`: produces canonical markdown artifact
- Groups comments by file, sorted by start line
- Resolved threads excluded (only open, actionable feedback)
- `fenceFor()`: computes backtick fence one longer than longest backtick run in snippet (CommonMark rule), minimum 3
- `anchorLabel()`: renders line reference with drift info (moved shows relocated range + original, outdated flagged)
- File-level comments (line 0) render as `file` not `L0`
- `langForExt()`: maps 20+ extensions to language identifiers
- Optional agent instructions section: curl example against `/api/comments/{id}/replies`

### Frontend (`web/src/`)
- **App.tsx** (715 lines): top-level state, repo/branch pickers, 3-column resizable layout (FileExplorer / DiffView / CommentsPanel), SSE subscription with focus/visibility fallback, uncommitted toggle, agent instruction copy, export modal trigger
- **FileExplorer.tsx** (188 lines): hierarchical file tree built from diff paths, directory compression (a/b/c when single-child chains), folder stats (reviewed/total), collapse, status markers (A/M/D/R)
- **DiffView.tsx** (571 lines): per-file diff with syntax highlighting, Changed/Full toggle, inline comment threads, drag-select line ranges, image before/after for raster images, SVG text/image toggle, file-level comments for binaries, auto-collapse for large/reviewed files, expand on jump-target
- **LazyFile.tsx** (51 lines): IntersectionObserver wrapper, scroll anchoring, estimated height for unmounted files
- **CommentThread.tsx** (146 lines): root comment + replies + reply composer, resolve toggle, edit/delete, moved/outdated badges
- **CommentsPanel.tsx** (75 lines): cross-file comment overview, jump-to with poll-for-mount
- **CommentComposer.tsx** (67 lines): type select (bug/suggestion/question/nit) + body textarea, reused for new/edit
- **ExportModal.tsx** (108 lines): markdown-it rendered preview (html:false), Raw toggle, copy/download, include-instructions checkbox
- **highlight.ts** (68 lines): Shiki with JS regex engine (not oniguruma — avoids browser wasm load failure), all ~235 languages, lazily fetched per file, alias resolution from Shiki metadata + extras map
- **api.ts** (130 lines): fetch wrappers, error parsing from JSON body, blob URL helper
- **types.ts** (99 lines): shared TypeScript types, `effectiveLines()` for current comment position, `lineLabel()`

### Build
- Frontend built to `web/dist` before Go build (embedded via `//go:embed all:web/dist`)
- Vite dev server proxies `/api` to Go server
- `web/package.json`: React 19, Shiki, markdown-it, Vite 7 + preserveGitkeep plugin
- Release workflow builds for darwin-arm64, linux-amd64, windows-amd64

## Key Techniques

### Drift-resistant anchoring
The project's core innovation: comments survive branch rebasing and new commits. Uses a two-tier system: (1) precise git line tracking via `git diff <commit_sha> head -- path` with `MapOldLine` hunks — catches genuine moves vs coincidental reappearances; (2) snippet matching as fallback for worktree comments, comments without commit_sha, and binary/renamed files. Each comment records the `commit_sha` it was anchored against (resolved at add time) and a `worktree` flag for which side to compare against.

### Derived state, never persisted
Comment staleness (`anchorStatus`, `currentStartLine`, `currentEndLine`) and reviewed-file validity are recomputed on every read, never trusted from stored flags. The fields exist on the `Comment` struct with `omitempty` JSON tags but the store never reads or writes them — they're filled by `annotateReview()` in the API layer. This means the reviewer always sees current truth even after force-pushes or rebases.

### Reviewed-files with content fingerprinting
When a file is marked reviewed, a SHA-256 fingerprint of its current content is captured alongside a `worktree` flag. On every review read, `annotateReview()` re-hashes the current content and drops any file whose fingerprint no longer matches. Changed files automatically revert to unread — no stale "reviewed" checkmarks after new commits.

### Coalescing SSE hub
The in-memory pub/sub hub uses size-1 buffered channels with non-blocking sends. A dropped ping is harmless: every ping makes the client refetch full canonical state, so at most one refresh is ever owed. Empty review entries are pruned on last unsubscribe. 25s keepalive comments keep streams warm and turn half-open connections into write errors.

### Backtick-safe markdown export
`fenceFor()` scans the snippet for the longest run of backticks, returns a fence one longer (minimum 3). Prevents a snippet containing triple-backtick code fences from prematurely closing the markdown code block — a real edge case when reviewing markdown files that contain code examples.

### Idempotent schema migration
`ensureColumn(table, column, ddl)` checks `PRAGMA table_info` for existing columns before `ALTER TABLE ADD COLUMN`. Each is a no-op once present, making schema additions safe across upgrades without a migration framework. Used for 7 columns added after the initial schema.

### Working tree diff with untracked files
`DiffWorktree` goes beyond `git diff <base>` to include untracked non-ignored files via `git ls-files --others --exclude-standard -z` (NUL-separated for safe path handling). Each untracked file gets a synthesized `FileDiff` with `newFileDiff()` — binary detection via NUL byte, empty files get no hunks, text files get a single "all added" hunk.

### Custom diff parser with content-safety
The `parseDiff` function correctly handles edge cases that trip up naive parsers: `---`/`+++` appearing in content (deleted SQL comments, added `++` lines) by checking hunk context before header matching. Also handles `\ No newline at end of file` escape lines, binary file detection, rename chains, and forces canonical `a/`/`b/` path prefixes regardless of user git config.

### Review resume regardless of export status
`CreateOrGetReview` matches on `(repo_path, base_ref, head_ref)` regardless of status — so exporting (which sets status to `exported`) never orphans an in-progress review. Re-opening the same branch resumes it with all comments intact.

## Design Decisions

### Git subprocess vs go-git library
**Chosen: shell out to git binary.** Correct on renames, submodules, `.gitattributes` diff filters, merge strategies. The trade-off is git must be installed (no pure-Go portability) and subprocess startup overhead. The right call for a tool whose entire purpose is reading git state — correctness on real repos beats theoretical portability.

### Single connection SQLite vs connection pool
**Chosen: `SetMaxOpenConns(1)`.** SQLite's `foreign_keys` pragma is per-connection — with a pooled DB some connections would miss it and skip `ON DELETE CASCADE`. Single connection makes the pragma authoritative and gives transactions full check-then-insert atomicity. DB access is low-frequency (git diffs/file reads don't hit SQLite), so serialization is free. A bold but correct call.

### Backend as source of truth vs client-owned state
**Chosen: backend is canonical, frontend caches.** Discrete actions (add/delete/toggle) save immediately. SSE hub broadcasts "changed" pings; clients refetch full state on ping. This means the backend can derive staleness, filter reviewed files, and compute anchor status on every read — the frontend never has to maintain its own truth. No conflict resolution, no optimistic update bugs beyond transient flickers.

### Two-level threads vs arbitrary nesting
**Chosen: comment (root) + replies (flat).** Comments have type, anchor, snippet, resolve; replies have only body + timestamp. FK cascades ensure replies never orphan. A conscious simplification — GitHub-style deep threading is complex to render and navigate. Two levels is enough for "reviewer says X, agent replies Y, reviewer acknowledges."

### Derived staleness vs write-time persistence
**Chosen: recompute on every read.** The alternative — update all comments on every push — is expensive and easily wrong. Recomputing on read means a freshly pulled branch always shows correct anchor status. The cost is O(comments × files/diffs) per review load, but reviews are human-scale (tens to hundreds of comments), not machine-scale.

### Ping-and-refetch SSE vs per-event payloads
**Chosen: simple `data: changed` pings only.** Clients refetch the full review. This means the SSE hub has no payload to serialize or version, no risk of desync between event types, and a dropped ping is at most one missed refresh. The alternative (sending full comment/reply state in SSE events) would mean duplicating serialization logic and keeping event payloads in sync with the REST API.

### JS regex engine for Shiki vs oniguruma WASM
**Chosen: JS regex with `forgiving: true`.** Avoids the browser WASM load failure that Shiki's default oniguruma engine triggers in some environments. `forgiving` skips the few oniguruma-only grammar patterns instead of throwing. The trade-off: some complex patterns (backreferences, lookbehind in certain grammars) degrade gracefully to unhighlighted tokens rather than breaking the whole page.

### Responsiveness patterns for large diffs
**Chosen: multiple defense layers.** `LazyFile` IntersectionObserver viewport-mounting avoids fetching/tokenizing off-screen files. Files > 500 changed lines auto-collapse. Files > 2000 lines skip syntax highlighting. Panel resize writes `grid-template-columns` directly to the DOM during drag (no React re-render per mousemove). A practical stack — not a single silver bullet, but layers that each cut a different cost.

### No authentication or multi-user support
**Chosen: local-only, single-user.** This is the defining constraint that makes the architecture simple. No auth middleware, no user table, no permissions. The server listens on 127.0.0.1 only. The trade-off is no team collaboration, but the purpose is "review before handing to agent," not "team code review."

## Comparison Notes

Unlike **[[OpenCodeReview]]** (Alibaba's AI code review CLI), which uses AI to *generate* review comments, local-review is a tool for *humans to write* review comments that are then handed to an AI coding agent. The direction is inverted: OpenCodeReview has AI reviewing human code; local-review has humans reviewing code for AI to act on.

Unlike **[[Bram]]**, which layers a two-gate approval workflow over Claude Code/Codex CLI sessions, local-review is purely a review artifact generator — it doesn't execute or approve anything. It produces markdown, which can feed into Bram, Claude Code, or any other agent.

Unlike **[[DeltaDB]]** (Zed's version control for the agent era), which traces every line of code to the conversation that produced it, local-review only cares about the review moment — a snapshot of what the human thought about the code at a particular point in time.

Unlike **[[Metis — ARM AI Security Code Review]]**, which uses tree-sitter for deterministic call-graph analysis, local-review makes no attempt to understand code semantics. It operates purely at the diff and line level, leaving all understanding to the human reviewer and the receiving coding agent.

The tech stack echoes **[[Memento]]** (Go + SQLite + SSE runtime), but for a different purpose: Memento is a knowledge layer over email; local-review is a review layer over git.

---

*Source: https://github.com/rosenbjerg/local-review*
*Analyzed: 2026-07-11*
*Codebase: ~1,100 lines Go (main.go + internal/), ~1,100 lines TypeScript (web/src/), ~200 lines CSS, ~300 lines tests*
*Dependencies: Go stdlib + modernc.org/sqlite; React 19 + Shiki + markdown-it + Vite*
