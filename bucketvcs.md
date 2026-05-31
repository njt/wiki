# bucketvcs

A Git server implemented as a single Go binary that stores all repository data directly in cloud object storage (S3, R2, GCS, Azure Blob). No database holds Git objects; no block volume grows with repo size. The bucket IS the repository. Speaks native Git protocol v2 over HTTPS and SSH, with LFS, OIDC-based CI auth, bundle-URI/pack-URI clone acceleration, protected refs, custom hooks, webhooks, and self-maintenance.

---

## Architecture

bucketvcs is a **single-binary Git server** (~460K lines of Go) with four clean architectural layers:

**1. Storage abstraction** (`internal/storage/objectstore.go`)

The `ObjectStore` interface provides Get, PutIfAbsent, PutIfVersionMatches (CAS), DeleteIfVersionMatches, List, multipart uploads, and signed URLs. Four backends implement it: local filesystem, S3-compatible (S3/R2/MinIO), Google Cloud Storage, and Azure Blob. All backends pass a shared conformance test suite.

**2. Repo manifest system** (`internal/repo/`)

Each (tenant, repo) pair has a root manifest stored at a well-known object key. It's a JSON document tracking packs, indexes (.bvom, .bvcg, .bvrd deltas), refs, bundles, and default branch. All mutations use CAS (`PutIfVersionMatches`) via `repo.Commit()` with exponential backoff retry (default 8 attempts, 5ms base). Two-phase: write a transaction record first, then CAS-swap the root. Orphaned tx records are garbage-collected.

The manifest has a **schema gate**: old binaries parse only `schema_version` and `min_reader_version` first, reject future-schema repos cleanly, THEN parse the full header. This prevents confusing type-change parse errors.

**3. Git protocol engines** (`internal/gitproto/`)

- **uploadpack** (`uploadpack/service.go`): Handles `git fetch`/`git clone`. Dispatches on protocol-v2 commands (ls-refs, fetch, bundle-uri). For fetch, parses wants/haves, validates reachability, then shells out to `git pack-objects` on a bare mirror clone, streaming the pack via sideband.
- **receivepack** (`receivepack/service.go`): Handles `git push`. Parses commands, runs connectivity check, enforces policy (protected refs/paths), runs pre-receive hooks (via bubblewrap sandbox on Linux), applies refs to bare mirror, repacks to canonical pack, builds indexes, CAS-commits the manifest, enqueues webhooks and post-receive hooks.

Both use a shared **reachability Set** (`internal/reachability/set.go`) that combines a base commit graph (.bvcg) + object map (.bvom) with a chain of incremental deltas (.bvrd), one per push. This enables pure-Go negotiation without calling git.

**4. Gateway & SSH** (`internal/gateway/`, `internal/sshd/`)

The `serve` command starts an HTTP gateway (smart-HTTP protocol) and an optional SSH server. They share the same ObjectStore, auth store, rate limiter, policy service, and hooks service. HTTP handles the standard Git paths; SSH handles `git-upload-pack`, `git-receive-pack`, and `git-lfs-authenticate`.

### Supporting subsystems

- **Mirror** (`internal/mirror/`): Maintains an on-disk bare git clone for git CLI operations (pack-objects, rev-list). Version-tracked, synced from object storage on demand, process-wide flock.
- **Auth** (`internal/auth/`): SQLite/Postgres/libsql-backed. Users, scoped tokens, SSH deploy keys, OIDC token exchange for keyless CI, per-IP rate limiting.
- **Importer** (`internal/importer/`): Round-trips a bare git repo from disk into bucketvcs storage: fsck, pack-objects, build indexes, upload, CAS-commit.
- **Maintenance & GC** (`cmd/bucketvcs/maintenance.go`, `cmd/bucketvcs/gc.go`): Repack, rebuild commit graph, generate bundles, compact reachability deltas, sweep orphans.

---

## Key Techniques

### Custom binary indexes instead of reusing Git's formats

bucketvcs uses three custom binary formats designed for random access from object storage:

- **`.bvom`** (Object Map): OID → (pack_id, offset), sorted by OID for binary search. 32-byte header, SHA-256 trailer.
- **`.bvcg`** (Commit Graph): Full DAG with parent edges and generation numbers. v2 format adds u32 generation per commit. O(log n) parent lookup via binary search. SHA-256 trailer.
- **`.bvrd`** (Reachability Delta): Records new commits + ref tip changes for one push. These chain together like a log-structured merge tree; maintenance compacts the chain.

### Generation-number heap for ancestor walks

`WalkAncestors` (`reachability/set.go:150-185`) uses a max-heap keyed on commit generation numbers. Pops highest-generation commits first, pushes parents. Standard commit-graph algorithm, implemented in pure Go.

### Lazy negotiation fast path

Before materializing the expensive on-disk mirror, `serveFetchLazyPath` (`uploadpack/service.go:829-979`) tries pure-Go negotiation via the reachability Set. If the shipping plan is empty (client already up-to-date), skips the mirror entirely — saving hundreds of milliseconds for large repos. Falls through to the mirror path on any error, so correctness is maintained.

### Hot-path stub pattern for bundle-URI freshness

When the bundle tip already matches the current ref tip (`uploadpack/service.go:619-627`), the code installs panic-stub closures for `IsAncestor` and `WalkBack`. The freshness evaluator returns "current" before reaching them. If a future refactor removes the early return, the panic fires immediately in tests — a clever defensive pattern.

### Byte-verified idempotent uploads

Canonical pack keys are based on git's SHA-1 of pack bytes. Delta compression is non-deterministic across repacks, so re-imports can produce different bytes that hash to the same pack_id. `uploadFileVerified` (`importer/importer.go:520-546`) handles this: on ErrAlreadyExists, it SHA-256s the stored object and compares to the local file. Byte-identical → safe (our .bvom offsets still valid). Different bytes → error.

---

## Design Decisions

### Shells out to git CLI instead of reimplementing packfile generation

The server calls `git pack-objects`, `git rev-list`, `git fsck`, etc. as subprocesses. This keeps the binary smaller and avoids reimplementing git's enormously complex packfile delta search. Trade-off: requires `git` on PATH and adds subprocess management surface.

### Optimistic concurrency over distributed locks

The root manifest uses CAS. On conflict, `repo.Commit` retries with jittered exponential backoff. No lock service needed. Write contention ceiling is the retry budget — fine for a Git server where push frequency per repo is seconds-to-minutes.

### Import is not fully atomic

Import creates an empty repo (version 1), uploads objects, then commits the body (version 2). A crash in the middle leaves an orphaned empty repo. The code explicitly documents this as a known limitation; GC sweeps orphans. A single-CAS create-with-body primitive would fix this.

### Base+delta reachability over full rebuilds

Each push produces a small .bvrd delta instead of rebuilding the full commit graph. This saves computation per push but requires periodic compaction (maintenance) and adds merge complexity. The delta chain is effectively a log-structured merge tree for commit metadata.

### Schema-gated forward compatibility

The manifest's two-phase parse means old binaries reject future-schema repos with a clear error message, not a confusing JSON type mismatch. This is a small detail that most projects get wrong.

---

## Comparison Notes

Unlike **Radicle** (P2P sovereign forge focused on decentralization via gossip), bucketvcs focuses on **storage economics** — bring your own bucket, pay per-GB, no vendor lock-in.

Unlike **Gitea/GitHub/GitLab** (traditional: repos on block volumes + metadata in databases), bucketvcs **eliminates the block volume entirely** for Git data. Only auth metadata touches a database (SQLite).

Shares the "object storage as source of truth" pattern with **Graft** (SQLite replicated via object storage), but applies it to Git repos rather than databases.

The custom binary index approach (.bvom, .bvcg, .bvrd) is reminiscent of how **Dolt** builds custom storage formats for Git-like operations on non-Git data.

The mirror system (bare clone on disk for git CLI operations) follows a **sidecar cache** pattern — the bucket is truth, the mirror is a performance optimization that can be rebuilt at any time.

---

Tags: #tool #project #git #storage #infrastructure #database

*Sources: [[raw/bucketvcs]]*
*Last updated: 2026-05-31*
