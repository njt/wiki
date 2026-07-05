---
url: https://github.com/erans/bucketvcs
title: bucketvcs — Git server backed by cloud object storage
author: Eran S. (erans)
date_fetched: 2026-05-31
date_published: 2025
---

# bucketvcs — Raw Analysis

## Project Summary

bucketvcs is a Git server implemented as a single Go binary that stores all repository data directly in cloud object storage (S3, R2, GCS, Azure Blob, or local filesystem). No database holds Git objects; no block volume grows with repository size. The bucket IS the repository. It speaks native Git protocol v2 over HTTPS and SSH, supports LFS, OIDC-based CI auth, protected refs, custom hooks, webhooks, bundle-URI/pack-URI acceleration, and background maintenance.

~460K lines of Go across ~200 source files. Dependency footprint is small: only `google/uuid` and `golang.org/x/mod` as direct non-cloud dependencies. Cloud SDKs are brought in for storage backends.

## Architecture Deep Dive

### Storage Layer (`internal/storage/`)

The central interface is `ObjectStore` (objectstore.go:32-96), providing:
- `Get`/`Head`/`GetRange` — reads
- `PutIfAbsent` — create-only (idempotent upload)
- `PutIfVersionMatches` — CAS update
- `DeleteIfVersionMatches` — conditional delete
- `List` — prefix listing with pagination
- `CreateMultipart`/`CompleteMultipartIfAbsent` — large object upload
- `SignedGetURL` — short-lived signed URLs for direct access

Four backends: localfs (filesystem), s3compat (S3/R2/MinIO), gcs (Google Cloud Storage), azureblob (Azure Blob Storage). Each passes a shared conformance suite (`internal/storage/conformance/`).

### Repo & Manifest System (`internal/repo/`)

Each (tenant, repo) pair has a well-known object key for its root manifest — a JSON document that tracks:
- `packs[]` — canonical pack entries with pack_id, key, size, object count
- `indexes{}` — references to .bvom (object map), .bvcg (commit graph), .bvrd (reachability deltas)
- `refs{}` — raw ref map (or reference to sharded refstore)
- `bundles[]` — bundle-uri entries for clone acceleration
- `default_branch` — HEAD target

**Manifest CAS**: All mutations use `PutIfVersionMatches` on the root manifest — optimistic concurrency with no distributed locks. `repo.Commit()` (repo.go:225-330) retries on version mismatch with exponential backoff (default 8 retries, 5ms base). Each attempt writes a fresh tx record first, then CAS-swaps the root. Orphaned tx records are cleaned by GC.

**Schema gate** (manifest/cas.go:26-81): The root manifest header carries `schema_version` and `min_reader_version`. `ReadRoot` parses only these two fields first, applies the gate, THEN parses the full header. This means a future-schema manifest where other header fields changed type still fails with `ErrUnsupportedSchema`, not a cryptic parse error.

### Custom Binary Indexes

Instead of relying on Git's native pack index format, bucketvcs uses three custom binary formats:

1. **`.bvom` (BucketVCS Object Map)** — `internal/objindex/`
   - Maps OID → (pack_id, offset_in_pack)
   - 32-byte header (magic "BVOM", version, count, pack_table offset)
   - Records sorted by OID for binary search
   - SHA-256 trailer

2. **`.bvcg` (BucketVCS Commit Graph)** — `internal/commitgraph/`
   - Full commit graph with parent edges and generation numbers
   - v1 format: OID(20) + n_parents(1) + parents[n]*20 — no generations
   - v2 format (M10): adds u32 generation_number after OID
   - Tips section maps ref names to commit OIDs
   - Supports binary search by OID for O(log n) parent lookups
   - SHA-256 trailer

3. **`.bvrd` (BucketVCS Reachability Delta)** — `internal/reachability/deltaindex/`
   - Incremental delta: one per push, recording new commits + ref tip changes
   - `Set` (reachability/set.go:20-26) merges base .bvcg + .bvom with all .bvrd deltas
   - Deltas chain in order; latest delta wins for OID and ref resolution
   - `refstore` subsystem stores refs inline in the manifest body or sharded to separate storage objects for large repos

### Git Protocol Implementation (`internal/gitproto/`)

**uploadpack** — handles `git fetch` / `git clone`:
- `Advertise()` sends capability advertisement (v0/v1/v2)
- `Service()` dispatches: command=ls-refs, command=fetch, command=bundle-uri
- `serveFetch()` (service.go:177-527): Parse wants/haves, validate reachability, run `git pack-objects` on the mirror bare repo, stream pack via sideband
- **Lazy negotiation fast path** (service.go:829-979): Before materializing the mirror, tries pure-Go negotiation via the reachability `Set`. If the shipping plan is empty (client up-to-date), skips the mirror entirely. Falls through to mirror path on any error.
- Bundle-URI: evaluates freshness (current/warm/stale), mints signed URLs for direct object-store download

**receivepack** — handles `git push`:
- Parses the push command list, validates connectivity
- Policy check (protected refs, protected paths)
- Custom hooks (pre-receive via bubblewrap sandbox on Linux)
- Applies refs to bare mirror, repacks to canonical pack, builds indexes
- Commits via CAS on root manifest
- Enqueues webhooks and post-receive hooks asynchronously

**v2proto** — shared protocol v2 utilities: `ls-refs`, `fetch` args parsing, bundle-uri response encoding, packfile-uris evaluation

### Gateway & SSH (`internal/gateway/`, `internal/sshd/`)

The `serve` command starts both an HTTP gateway and an optional SSH server. They share:
- The same `ObjectStore` (via --store URL)
- The same auth store (SQLite/Postgres via --auth-db)
- The same rate limiter, policy service, webhook service, hooks service

HTTP gateway maps Git's smart-HTTP protocol paths (`/info/refs`, `/git-upload-pack`, `/git-receive-pack`) to the engine. SSH server handles `git-upload-pack`, `git-receive-pack`, `git-lfs-authenticate` commands.

### Mirror System (`internal/mirror/`)

Maintains an on-disk bare git clone (the "mirror") for `git pack-objects` and other git CLI operations. Version-tracked via a sentinel file. Syncs from object storage on demand, acquiring a flock for process-wide mutual exclusion. Separate mirror instances for HTTP and SSH to avoid lock contention.

### Auth System (`internal/auth/`)

SQLite-backed (with optional Postgres and libsql backends). Handles:
- Users, tokens (scoped, revocable), SSH deploy keys
- OIDC token exchange (RFC 8693) for keyless CI
- Rate limiting on credential failures (per-IP burst bucket)
- Permission model: per-repo permissions with token scopes

### Importer (`internal/importer/`)

Round-trips a bare git repo from local disk into bucketvcs storage:
1. Clone source as bare mirror
2. `git fsck` (connectivity check)
3. `git pack-objects --all` to create a canonical pack
4. Build .bvom and .bvcg from the pack
5. Upload pack, idx, .bvom, .bvcg via PutIfAbsent
6. Repo.Create (claim) → upload → Repo.Commit (CAS body)
7. UploadFileVerified for idempotent re-imports (byte-identity check on SHA-1 collision)

### GC & Maintenance

- **GC** (`cmd/bucketvcs/gc.go`): Sweeps orphan tx records, unreferenced packs/indexes, old bundles
- **Maintenance** (`cmd/bucketvcs/maintenance.go`): Repacks, rebuilds commit graph, generates bundles, compacts reachability delta chain
- **Reshard refs** (`cmd/bucketvcs/reshard_refs.go`): Converts inline refs (in manifest JSON) to sharded ref storage for repos with many refs

## Key Techniques

### Generation-number heap for ancestor walks
`WalkAncestors` (reachability/set.go:150-185) uses a max-heap keyed on commit generation numbers to walk ancestors in generation-descending order. This is a standard commit-graph algorithm reimplemented in pure Go without calling git.

### Commit-graph generation numbers via memoized DFS
`computeGenerations` (commitgraph/build.go:58-101) computes `gen(c) = 1 + max(gen(parents))` with memoization and cycle detection via a `visiting` set. Roots get gen=1. The `visiting` map detects cycles (shouldn't happen in valid git repos, but defensive).

### Delta chain for incremental reachability
Instead of rebuilding the full commit graph on every push, each push produces a `.bvrd` delta. The `Set.Load()` (reachability/set.go:30-101) merges the base graph with all deltas in order. `deltaIndex` is a flat map from OID to `CommitRecord` — latest delta wins for each OID. Ref tips also layer: each delta's ref tips overwrite the base map, with zero OID meaning deletion.

### Hot-path stub pattern for bundle-URI freshness
When the bundle tip already matches the current ref tip, `serveBundleURI` (service.go:619-627) installs panic-stub closures for `IsAncestor` and `WalkBack`. The freshness evaluator returns "current" before ever calling them. If a future refactor removes the early return, the panic fires immediately in tests.

### CSV-split env var layering
Storage backends use environment variables as an overlay on parsed URL config (store.go:132-185). `BUCKETVCS_S3_REGION` takes precedence over `AWS_REGION`, giving operators explicit control without breaking standard SDK env var behavior.

## Design Decisions & Trade-offs

### Shells out to git CLI
The server calls `git clone`, `git pack-objects`, `git fsck`, `git rev-list`, `git update-ref`, etc. as subprocesses. This keeps the Go binary smaller and avoids reimplementing git's packfile generation (which is enormously complex). The trade-off: `git` must be on PATH, and subprocess management adds error-handling surface.

### Optimistic concurrency over distributed locks
The root manifest uses CAS (`PutIfVersionMatches`). On conflict, `repo.Commit` retries with exponential backoff. This is simple, correct, and requires no lock service. The write-contention ceiling is the CAS retry budget (default 8 attempts × 5ms base × jitter) — fine for a Git server where push frequency per repo is measured in seconds-to-minutes, not milliseconds.

### Not-fully-atomic import
Import first creates an empty repo (version 1), then uploads objects, then commits the body (version 2). A crash between create and commit leaves an orphaned empty repo. The design doc (importer.go:292-315) explicitly acknowledges this as a known M2 limitation. GC sweeps orphaned repos.

### UploadFileVerified for idempotency
Canonical pack keys are based on git's SHA-1 of the pack bytes. Delta compression is non-deterministic across repacks (varies with threads, memory pressure), so a re-import can produce different bytes that hash to the same pack_id. `uploadFileVerified` (importer.go:520-546) detects this: on ErrAlreadyExists, it SHA-256s the stored object and compares to the local file. Byte-identical → safe; different bytes → error (our .bvom offsets would be wrong).

### Base+delta architecture for reachability
The .bvrd delta chain avoids rebuilding the full commit graph per push. Each push produces a small delta recording only the new commits and ref tip changes. `maintenance` compacts the chain back to a clean base. This is essentially a log-structured merge tree applied to commit-graph metadata.

### Schema-gated forward compatibility
The root manifest's two-phase parse (compat header → schema gate → full parse) means old binaries reject future-schema repos cleanly. This is a thoughtful detail that most projects get wrong (producing confusing parse errors instead of a clear "unsupported schema version" message).

## Comparison to Related Projects

- **Radicle**: P2P sovereign code forge on Git. Radicle focuses on decentralization via gossip protocol and cryptographic identity; bucketvcs focuses on storage economics via bring-your-own-bucket.
- **Gitea/GitHub/GitLab**: Traditional architecture: Git repos on block volumes, metadata in databases. bucketvcs eliminates both for Git data — only auth metadata uses SQLite/Postgres.
- **Dolt**: Git-like versioning for SQL databases. Different domain (data, not code) but shares the "Git semantics without a traditional Git server" concept.
- **Graft**: SQLite replicated via object storage. Shares the "object storage as source of truth" pattern but for databases, not Git repos.

## File Structure Summary

```
cmd/bucketvcs/          — CLI entry point (main.go), subcommands (serve, import, export, gc, user, token, repo, etc.)
internal/
  auth/                 — Auth types, tokens, permissions, scopes, rate limiting
  auth/sqlitestore/     — SQLite/Postgres/libsql auth backend with migrations
  commitgraph/          — .bvcg binary format: build, read, format
  gateway/              — HTTP gateway (smart-HTTP protocol, bundle/pack proxying, LFS API)
  gitcli/               — Git CLI wrappers (pack-objects, rev-list, fsck, clone, bundle)
  gitproto/
    receivepack/        — Git receive-pack protocol (push): advertise, service, engine, delta upload, policy
    uploadpack/         — Git upload-pack protocol (fetch/clone): advertise, service, engine, negotiate
  hooks/                — Tier 3 hooks (pre-receive, post-receive) with bwrap sandbox
  importer/             — Import bare git repo into bucketvcs storage
  lfs/                  — Git LFS: batch API, token issuance, quotas, locks
  mirror/               — On-disk bare git mirror management with flock
  objindex/             — .bvom binary format: OID → pack location
  oidc/                 — OIDC token verification
  pack/                 — Git packfile reader, OID types
  pktline/              — Git pkt-line protocol framing + sideband
  policy/               — Protected refs and paths enforcement
  reachability/         — Unified reachability Set (base + deltas), WalkAncestors
  reachability/deltaindex/ — .bvrd binary format for incremental reachability deltas
  repo/                 — Repo handle: Open, Create, Commit (CAS with retry)
  repo/keys/            — Storage key generation for repos
  repo/manifest/        — Root manifest: header, body, CAS, schema gate
  repo/refstore/        — Ref storage: inline (manifest body) or sharded
  repo/tx/              — Transaction records and commit markers
  sshd/                 — SSH server: git-upload-pack, git-receive-pack, git-lfs-authenticate
  storage/              — ObjectStore interface + conformance test suite
  storage/localfs/      — Local filesystem backend
  storage/s3compat/     — S3/R2/MinIO backend
  storage/gcs/          — Google Cloud Storage backend
  storage/azureblob/    — Azure Blob Storage backend
  v2proto/              — Git protocol v2: ls-refs, fetch args, bundle-uri, pack-uri
  webhooks/             — Webhook delivery with retry
```
