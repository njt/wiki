---
url: https://github.com/telepath-computer/stash
title: Stash — Conflict-Free Folder Sync
author: Telepath Computer
date_fetched: 2026-06-11
date_published: 2026
topics:
  - misc
---

# Stash — Full Technical Analysis

## What it is

A TypeScript CLI tool (~3,200 lines of source, ~6,900 total with tests) that syncs any folder across machines using GitHub as a backend. It's Dropbox-meets-git: files sync automatically, text edits merge conflict-free via Google's diff-match-patch, and binary files use last-modified-wins. The project is by the team at Telepath Computer, published as `@telepath-computer/stash` on npm (v0.4.1).

## Repository Structure

```
src/
  stash.ts (947 lines)       — Core engine: scan, reconcile, merge, snapshot
  cli.ts (10 lines)           — Entry point
  cli-main.ts (868 lines)     — Commander CLI, inquirer prompts, service mgmt
  watch.ts (388 lines)        — Debounced FS watcher + periodic polling loop
  daemon.ts (282 lines)       — Background service managing multiple Watch instances
  providers/
    github-provider.ts (522)  — GitHub REST + GraphQL API transport
    index.ts                  — Provider registry
  ui/                         — Terminal rendering (sync-renderer, watch-renderer, live-line)
  utils/                      — fs, hash, text, is-tracked-path
  types.ts (70 lines)         — Core type definitions
  errors.ts (38 lines)        — Custom error classes
  emitter.ts (1 line)         — Re-exports @rupertsworld/emitter
  migrations.ts (139 lines)   — Schema migration for .stash/ layout changes
docs/
  architecture.md, sync.md, reconciliation.md, config.md
  providers/ (overview.md, github.md)
tests/
  unit/ (18 test files), integration/ (5), e2e/ (2), helpers/ (5)
```

## Dependencies

- `diff-match-patch` — Google's text diff/patch algorithm (same as Obsidian Sync)
- `@parcel/watcher` — Native filesystem events (inotify/FSEvents/ReadDirectoryChanges)
- `@rupertsworld/daemon` — OS service installation (launchd, systemd)
- `@rupertsworld/emitter` — Lightweight event emitter
- `@rupertsworld/disposable` — Resource cleanup abstraction
- `@inquirer/prompts` — Interactive CLI prompts
- `commander` — CLI argument parsing

Zero database dependencies. All state is files on disk: JSON snapshots and text-base copies.

## Architecture

### Layered Design (5 tiers)

1. **CLI layer** (`cli-main.ts`) — Commander-based parsing with inquirer prompts. Thin — delegates to core after argument resolution.
2. **Watch/Daemon layer** (`watch.ts`, `daemon.ts`) — Reusable scheduling with debounce, poll fallback, and multi-stash management.
3. **Core engine** (`stash.ts`) — The 947-line heart: scanning, reconciliation via merge table, three-way text merge, snapshot management, drift detection, push-before-write ordering.
4. **Provider transport** (`providers/`) — GitHub REST API + GraphQL for blob metadata. Stateless across calls; snapshot is the source of truth.
5. **Infrastructure** — `@rupertsworld/daemon` for OS service lifecycle, `@parcel/watcher` for native FS events.

### Boundary Rules (enforced by architecture.md)

- CLI is thin over Stash core
- Watch is headless — no stdin/TTY rendering
- Providers are transport-only — no merging, no local disk access
- UI helpers (src/ui/) are presentation-only — no sync logic

### Sync Lifecycle (8 steps)

1. Read local `snapshot.json`
2. `scan()` — walk filesystem, hash every tracked file, diff against snapshot → `ChangeSet`
3. `provider.fetch(localSnapshot)` — get remote `ChangeSet`
4. `reconcile(local, remote)` — apply merge table → `FileMutation[]`
5. `computeSnapshot()` — next snapshot from old + mutations
6. `buildExpectedHashes()` + `hasAnyPathDrift()` — retry if local files changed during fetch
7. `provider.push(payload)` — write files, deletions, and new snapshot; throws `PushConflictError` on remote divergence
8. `apply()` — deletes before writes (case-insensitive FS safety), re-check drift per file, emit mutations

### Sync Locking

Two guards:
- In-process: `syncInFlight` boolean on Stash instance
- Cross-process: `.stash/sync.lock` created atomically (`wx` flag), contains `{pid, startedAt, hostname}`
- Stale lock recovery: locks >10 minutes are reclaimed
- Release in `finally` block

## Key Techniques

### Snapshot-Based Diffing (not history-based)

Rather than tracking a commit DAG, Stash stores a flat JSON dictionary of `{path: {hash}}` entries. Every tracked file's SHA-256 hash is recorded. Changes are detected by comparing current filesystem state against this snapshot. This is vastly simpler than git's object model but loses history — recovery is only possible from the GitHub remote's commit log.

### Three-Way Text Merge via diff-match-patch

When both sides modified the same text file, Stash doesn't pick a winner or show conflict markers. Instead:
1. Load the snapshot base (from `.stash/snapshot/<path>`)
2. Compute `patch_make(base, local)` — what local changed
3. Apply it: `patch_apply(localPatches, base)` → intermediate
4. Compute `patch_make(base, remote)` — what remote changed
5. Apply it: `patch_apply(remotePatches, intermediate)` → merged result

If the merged result matches one side exactly, that side's write is skipped (optimization). If no snapshot base exists (first sync), falls back to two-way merge.

### Push-Before-Write Ordering

The sync pushes remote changes BEFORE applying local disk writes. This is the opposite of what you might expect, but it ensures:
- If push fails, local files are untouched
- If push succeeds but a later apply step fails or is skipped (drift), the next sync self-heals from the remote + preserved snapshot base

### Dual Drift Detection

Drift is checked at TWO points per cycle:
1. **Pre-push**: re-check only mutation-targeted paths (not full rescan). If any drifted → restart cycle.
2. **Per-file post-push**: before each `disk: "write"` mutation, re-check that file's hash. If drifted → skip write, roll back that path's snapshot entry to previous base, so next sync treats both sides as changed and re-merges.

This preserves newer local edits without rolling back the remote push that already succeeded.

### GitHub GraphQL Batch Blob Classification

The GitHub provider uses a single GraphQL query to batch-check `isBinary` and `text` for all candidate files at once — rather than N individual REST calls. This avoids rate limiting and dramatically reduces latency:

```graphql
query {
  repository(owner: "...", name: "...") {
    f0: object(expression: "main:path/to/file0") { ... on Blob { text isBinary } }
    f1: object(expression: "main:path/to/file1") { ... on Blob { text isBinary } }
    ...
  }
}
```

### Empty Repo Bootstrap

GitHub's Git Data API returns 409 on empty repos (no commits). The provider detects this and bootstraps by creating `.stash/snapshot.json` via the Contents API first, then polls the branches endpoint (10 attempts, 300ms apart) until `main` resolves.

### Case-Insensitive Filesystem Safety

Three measures prevent case-only renames from causing data loss:
- **Drift checks require exact path casing** for each segment
- **Deletes happen before writes** during `apply()` (sorted mutations)
- **`ensureDirectoryCasing()`** renames directories whose casing differs from the mutation path before writing files into them

### File Tracking via Segment Rules

`isTrackedPath()` at `src/utils/is-tracked-path.ts` implements the tracking policy as a pure function: a path is excluded if any segment starts with `.` (dotfiles, dot-directories, `.stash/`), is `.` or `..`, or is a symlink. This is shared between scanning and watching for consistency.

### Sync Log with Byte Cap

The daemon persists per-stash logs (`.stash/sync.log`) capped at 1MB. When a new line would exceed the limit, it shifts lines from the front — a ring-buffer-in-a-file pattern.

### Retry with Bounded Attempts

Sync retries on two conditions: pre-push drift and `PushConflictError`. Capped at 5 attempts. No unbounded restart loop — if the limit is exhausted, the last error propagates.

## Design Decisions

### Optimized for: simplicity and reliability
- One connection per stash (enforced; throws `MultipleConnectionsError` otherwise)
- Flat JSON snapshot, not a commit DAG
- Text-merge-first philosophy: structured data (JSON, YAML) is merged as text, not parsed — "we will add strict structured data parsing if this use case proves important"
- Provider-agnostic but provider contract is opinionated about snapshots

### Sacrificed: local version history
- No local changelog or undo stack
- Recovery depends on the GitHub remote's commit history
- Binary "merge" is just last-modified-wins with a timestamp comparison

### Git repo coexistence: guarded, not blocked
- By default, refuses to sync directories containing `.git/` — branch switches look like mass file edits
- Configurable via `stash config set allow-git true`
- Recommendation: git locally, stash manually, disable background sync while using git

### Transport-agnosticism as architectural discipline
Stash itself is completely provider-agnostic — the `Provider` interface is `fetch()`, `get()`, `push()`. But the only production provider is GitHub. The abstraction is clean enough that adding S3 or WebDAV would be straightforward, though binary detection consistency across providers is a real challenge (the docs explicitly warn about ping-pong if a provider classifies files differently than the local scanner).

### Background daemon as OS service
Rather than running as a user process, `stash start` installs itself via `@rupertsworld/daemon` (launchd on macOS, systemd on Linux). This means it survives reboots. The daemon watches the global config file for changes via `@parcel/watcher`, hot-reloading its stash registry when new stashes are added or removed.

## Comparison to Related Approaches

- **vs git**: Stash auto-syncs without explicit commits; merges text without conflict markers; uses GitHub as dumb storage, not as a VCS. But lacks local history, branching, or blame.
- **vs Dropbox/iCloud**: Stash merges concurrent text edits rather than last-writer-wins for everything. But requires a GitHub account and repo, and doesn't handle selective sync or file sharing.
- **vs Obsidian Sync**: Both use diff-match-patch for three-way merge (Stash explicitly calls this out in its README). But Stash is tool-agnostic, open-source, and uses your own GitHub repo rather than Obsidian's servers.
- **vs Syncthing**: Syncthing is P2P, Stash is hub-and-spoke (GitHub is the hub). Syncthing handles any file type the same way (last-modified-wins); Stash does text-aware merging.
- **vs Resilio Sync**: Resilio is proprietary, P2P, and also last-modified-wins. Stash is open-source, centralized, and text-aware.

## Sharp Take

Stash is a beautifully disciplined piece of software. The code is clean, the documentation is thorough, and the architectural boundaries are explicit. It does one thing — sync folders via GitHub — and does it with unusual care for edge cases (case-insensitive filesystems, drift detection, stale lock recovery, empty repo bootstrapping).

**The hidden insight**: Stash's real innovation isn't GitHub-as-backend or diff-match-patch merging — it's the snapshot-based diffing model that makes the provider contract so simple. By reducing "what changed" to a flat JSON hash map diff, Stash avoids the complexity of CRDTs, operational transforms, or version vectors. The cost is losing local history, but for the use case of "sync my agent's working files across machines," that's the right trade.

**The risk**: The project is at v0.4.1 and depends on `@rupertsworld/*` packages which are also early-stage. The single-branch (`main`) model with force-push=false means GitHub branch protection rules could block syncs. And the text-only merge of structured data is a footgun waiting to happen if someone syncs a JSON config file from two machines simultaneously.
