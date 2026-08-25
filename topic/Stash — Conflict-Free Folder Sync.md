# Stash — Conflict-Free Folder Sync

Open-source CLI tool that syncs any folder across machines via a GitHub repo you own — no Daemon, no SaaS, just a TypeScript binary that merges text edits conflict-free using Google's diff-match-patch (the same algorithm Obsidian Sync uses). Changes appear in seconds via native filesystem watching or 10-second polling. Made by [Telepath Computer](https://telepath.computer/).

> "Sync any folder anywhere, conflict-free. Keep your agent memory, skills, and documents in sync across machines, agents, and collaborators."
> — README

## Architecture

Stash is a layered TypeScript application (~3,200 lines of source) structured as five tiers:

```
CLI (Commander + Inquirer) → Watch/Daemon → Core Engine → Provider Transport → OS Service
```

**Core engine** (`src/stash.ts`, 947 lines) owns scanning, reconciliation, merging, snapshot management, and the sync lifecycle. It's the heart of the system — the CLI and Watch layers are thin wrappers.

**Provider layer** (`src/providers/github-provider.ts`, 522 lines) implements the GitHub transport using both REST and GraphQL APIs. It never merges files or touches local disk — it's pure transport. The provider contract (`fetch`, `get`, `push`) is clean enough that S3 or WebDAV providers could be plugged in without touching core logic.

**Watch system** (`src/watch.ts`, 388 lines) combines native FS events (`@parcel/watcher`) with a 10-second poll fallback. It uses a 1-second debounce window and a state machine (idle → debouncing → syncing) to batch rapid changes. If a change arrives mid-sync, it queues a re-sync.

**Background daemon** (`src/daemon.ts`, 282 lines) manages multiple Watch instances — one per registered stash directory. It installs as an OS service (launchd on macOS, systemd on Linux) via `@rupertsworld/daemon` and hot-reloads when the global config file changes.

**Sync lifecycle** follows this order: scan local files against snapshot → fetch remote changes → reconcile into mutations → compute next snapshot → check for pre-push drift → push to remote → apply local writes (with per-file drift re-check). Push happens BEFORE local writes so a failed push never corrupts local state.

## Key Techniques

### Snapshot-based diffing, not history-based

Every tracked file's SHA-256 hash is stored in `.stash/snapshot.json`. Changes are detected by hashing current filesystem state and diffing against the snapshot — no commit DAG, no version vectors. This makes the provider contract dramatically simpler than git's object model. The trade-off: no local version history (recovery depends on GitHub's commit log).

### Three-way text merge via diff-match-patch

When both local and remote modified the same text file, Stash doesn't show conflict markers. It loads the snapshot base from `.stash/snapshot/<path>`, applies local's patches to the base, then applies remote's patches on top. If the result matches one side exactly, that side's write is skipped. This is how Obsidian Sync works — Stash is the open-source, tool-agnostic version of that approach.

### Dual drift detection

Filesystem drift is checked at TWO points per sync cycle, not just one:
1. **Pre-push**: re-hash only mutation-targeted paths. If any drifted → restart the whole cycle (up to 5 retries)
2. **Per-file post-push**: before each local write, re-check that file. If it drifted after the push succeeded → skip the write but roll back that path's snapshot entry so the next sync sees both sides as changed and re-merges

This means in-flight local edits are never silently overwritten, even if they happen mid-sync.

### GitHub GraphQL batch blob classification

Rather than N individual REST calls to check file types, the GitHub provider builds a single GraphQL query with aliased `object(expression: "main:<path>")` fields. This batch-checks `isBinary` and `text` for all candidate files in one request — avoiding rate limits and cutting latency.

### Empty repo bootstrapping

GitHub's Git Data API returns 409 on repos with no commits. The provider detects this and bootstraps by creating `.stash/snapshot.json` via the Contents API, then polls the branches endpoint (10 retries, 300ms intervals) until `main` resolves. This is a subtle but essential detail — without it, new repos would be permanently broken.

### Case-insensitive filesystem safety

Three measures prevent case-only renames from causing data loss on macOS/Windows: drift checks require exact path casing per segment, deletes always happen before writes in `apply()`, and `ensureDirectoryCasing()` renames directories whose casing doesn't match the expected path before writing files into them.

### Sync lock with stale recovery

Cross-process sync exclusion uses atomic file creation (`wx` flag) on `.stash/sync.lock`. Locks older than 10 minutes are treated as stale and reclaimed — preventing a crashed process from permanently blocking syncs. The lock file records `{pid, startedAt, hostname}` for diagnostics.

## Design Decisions

**Optimized for simplicity over history**: Flat JSON snapshots instead of a commit DAG. No local undo, no changelog — the GitHub remote IS the backup. For "sync my agent's files across machines," this is exactly right.

**Text-merge-first, even for structured data**: JSON and YAML files are merged as text, not parsed. The docs acknowledge this could produce invalid structured output and say they'll add parsing "if this use case proves important." This is a deliberate punt — simple and correct for prose, risky for config files.

**Git repo coexistence is guarded, not blocked**: By default, Stash refuses to sync directories containing `.git/` because branch switches look like mass file edits. Configurable via `stash config set allow-git true`, but the recommendation is to use git locally and stash manually.

**One connection per stash, enforced**: The core throws `MultipleConnectionsError` if you try to add a second connection. This simplifies reconciliation (no multi-remote merge) at the cost of flexibility — you can't sync one folder to two remotes.

**Push-before-write ordering**: Counter-intuitive but correct. Remote changes are pushed before local disk writes are applied. If push fails, nothing local was touched. If push succeeds but a write fails, the next sync self-heals.

## Comparison Notes

- **vs git**: Auto-syncs without explicit commits; merges text without conflict markers. Lacks local history, branching, blame.
- **vs Dropbox/iCloud**: Merges concurrent text edits (not just last-writer-wins). Requires a GitHub account and repo.
- **vs Obsidian Sync**: Same merge algorithm (diff-match-patch), but tool-agnostic and uses your own GitHub repo — no Obsidian lock-in.
- **vs Syncthing**: Hub-and-spoke (GitHub) not P2P. Text-aware merging vs Syncthing's uniform last-modified-wins.
- **vs the other Stash**: There's a different Go-based project also called "Stash" ([[Stash]]) that does AI agent memory with PostgreSQL+pgvector. Completely unrelated — name collision only.

## Tags

#tool #sync #files #merge #conflict-free #typescript #github #cli #open-source

## Cross-References

- [[Git Diff Drivers]] — git's external diff driver interface; Stash uses diff-match-patch instead
- [[Software Engineering Craft]] — the craft hub; Stash is an exemplar of clean architecture with explicit boundary rules
- [[Obsidian Introduction]] — Obsidian Sync uses the same diff-match-patch merge algorithm
- [[Copyparty]] — the inverse philosophy: no sync at all, serve files as-is from one canonical folder

---
Source: https://github.com/telepath-computer/stash (Telepath Computer, 2026). Fetched 2026-06-11.
