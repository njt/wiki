# Twigg

Twigg is an open-source, from-scratch reimplementation of Google's internal code-review platform **Critique** — the trunk-based, stacked-commit workflow Google engineers use daily. The interesting part isn't that it's "another code review tool"; it's that Twigg **replaces Git entirely** rather than bolting review onto it. By making stacked commits, versioned amend/rebase, hierarchical `OWNERS`, and path-triggered CI native to the version-control system itself, it turns a workflow that normally requires discipline and a pile of external tools into the default. Written in Go with SQLite everywhere, it's also a striking real-world instance of the [[The GUS Stack — Go, Unix, SQLite]] thesis.

#tool #project #vcs #code-review #git #go #sqlite #trunk-based-development #stacked-commits

---

## Architecture

A monorepo of **five binaries** sharing a VCS core, built entirely around a SQLite + append-only-log storage engine.

**The storage engine (`data/`)** is the heart. Blob bytes live in an append-only "datastrip" log (`data/appendlog`, tiered across files via `data/appendlog/tiered`, fixed-size blocks via `fileblock`); metadata lives in SQLite (`data/sqlitehelper` for migrations). `data/blobdb/db.go` is a content-addressed blob store whose `SetBlob`/`GetBlob` write each new blob *version* to the log and record `Offset`, `CompressedSize`, `Encoding`, and `DistanceToNonDelta` in the `sqlarge_blobs` table (webdb migration `0000000_setup.sql`). The same engine runs **on both server and CLI client** — the client's `.twigg/sqlarge.db` is a full local replica of the object store, not a shallow checkout.

**The VCS core (`twigg/`)** is the logic: `commit` (the commit type and its lifecycle state machine), `tree` (the SHA-256 Merkle tree and diff iterators), `server` (trunk state + submit/pull/push), `repo`, `workdir`, and `xchange` (the client↔server wire format, gob-encoded).

**The servers.** `twigg-web` is a Go server-rendered app (vendored `maragudk/gomponents` HTML builder + `pongo2` templates), with a `handlers/` tree for every page and a `services/` layer (review, owners, mirror, jobs, cicdpublisher, stripe, oidc). `twigg-track` schedules jobs via `squeue`, a SQLite-backed queue whose `squeue/sqlock` wraps every transaction in a `sync.RWMutex` — one writer at a time per database, a deliberate choice for a single-node server. `twigg-runner` executes jobs in Docker or LXD. `tw` is the CLI.

## Key techniques

**Versioned commits instead of a hash DAG.** This is the load-bearing idea. A `Commit` has a **local ID** (a small monotonic integer per repo) *and* a **version number**; it also carries `ServerL`/`ServerV` for its global identity (`twigg/commit/commit.go`). Amending, rebasing, or submitting does **not** create a new commit — it creates `newVersion` of the *same* commit (`L` unchanged, `Version+1`) and marks the old version `Obsolete` with a successor pointer and a `BirthReason` (Commit, Amend, ManualRebase, AutoRebaseOfChildren, Submit, Restore). Git rewrites history by minting new hashes and orphaning old commits; Twigg keeps a stable commit DAG whose *nodes have version histories*. That's what makes "review version by version" possible — a reviewer can diff v1→v2 of the same logical change — and it's why commits can be addressed as `C`, `CvV`, or `c/CvV` (`twigg/cli/app.go`).

**Trunk-based submit = a rebase.** The server holds a single `Top_` commit (`twigg/server/server.go`). `Submit` (`twigg/server/submit.go`) either creates a *trivial submit* if the parent is already the trunk tip, or runs a three-way tree rebase onto `Top_` and creates a rebase-submit commit. There is no merge commit; every submit advances the one trunk head, and you must submit in order (parent submitted first). Stacked commits fall out of this for free.

**Lockstep tree diffing.** Diffs are produced by a *parallel iterator* over two Merkle trees (`twigg/tree/walk2.go`): it walks both trees in lockstep, comparing lexicographically sorted paths at equal depth, and `SkipChildrenOnNext()` prunes any unchanged subtree — so the diff cost is O(changed paths), not O(all files). `Walk4` compares two version-pairs (for commit-vs-parent × version-vs-version review). Line-level diffs use Go's stdlib **anchored/patience diff** (`twigg/diff/internal_diff.go`, copied verbatim from `go/src/internal/diff`) — O(n log n) via Szymanski's algorithm, chosen because unique-line anchoring produces clearer code-review diffs than Myers.

**Adaptive delta + flate blob compression.** `data/deltastream/front.go` lazily buffers up to 1 MB of the incoming blob while deciding the codec: small files (500 B–1 MB) get `godelta` (a vendored fork of `balacode/go-delta` in `data/balacode/go-delta`) delta-compressed against the parent version; larger or parentless blobs fall back to flate (deflate). `data/blobdb/db.go` caps consecutive deltas at `maxConsecutiveDeltaEncoded = 10`, then forces a full re-encode so decompression never walks an unbounded chain.

**Path-local CI/CD.** CI/CD is configured by `CI.json`/`CD.json` files that can live *anywhere* in the tree (`twigg-runner/runnerlib/parse.go`). On submit, `cicdpublisher/cicdrun.go` uses `SearchFileInChangedDirs` to find only the config files inside directories the commit actually touched, then runs only jobs whose `On` trigger (`push`/`submit`/`manual`) matches. Co-locating CI with the code it builds and running only affected jobs is the Bazel idea applied to CI. The parser is strict: `DisallowUnknownFields`, exactly-one-timeout-unit, max template recursion of 100, and only two step templates (`GetCode` expands to `tw init/key/server/pull` — the jobs fetch code with the Twigg CLI itself).

**Owners as a tree walk.** `twigg-web/services/owners/owners.go` checks each changed path by walking *up* the directory tree for `OWNERS` files; a path is approved if any LGTM-er appears in an OWNERS file at or above it, and an org's "supreme leaders" can override. Review state itself is version-aware: threads attach to a specific *commit version* and LGTMs are version-stamped, so `AddLgtm` rejects an LGTM at an older version than the current one (`services/review/service.go`).

## Design decisions

**Server-allocated integer IDs over content hashes.** Twigg gives up Git's decentralized, hash-identified commits in exchange for human-friendly `c/1341` and the `CvV` version syntax. The cost is real: a server must mint IDs, and identity isn't self-certifying. The payoff is the entire review workflow — versioned amends, version-anchored comments — that Git's immutability makes awkward.

**SQLite as the only database, everywhere.** `modernc.org/sqlite` (pure Go, no cgo) backs commits, reviews, blob metadata, and the job queue. `sqlock`'s single-writer mutex reveals the assumption: **one node per server**, no distributed consensus. That's a fine trade for "closed teams" but caps horizontal scaling — the opposite bet from [[bucketvcs]], which keeps Git's protocol but makes object storage the scale story.

**Snapshot Git mirror, not a translation.** `twigg-web/services/mirror/mirror.go` doesn't map Twigg history onto Git history; on each push it does a fresh `git init`, flattens the top commit, and pushes one commit carrying a `Twigg mirror c/CvV` marker for dedupe. It's an exit hatch (backup, switch-back), not a compatibility layer. The security care is notable: `GIT_ALLOW_PROTOCOL=https:http` blocks `ext::`/`file://`/`ssh://` RCE/SSRF vectors.

**Boring, vendored deps.** Stripe for billing, zitadel/oidc for login, pongo2 for templating, gomponents vendored in-tree. Nothing exotic — consistent with the "radical transparency, no security through obscurity" README stance, which is also why runbooks and deploy configs live in the repo.

## Comparison notes

**vs. Git (and [[Git Rebase for the Terrified]]).** Twigg's bet is that the pain of stacked commits, trunk-based development, and rebase-on-submit is not incidental to Git but *structural* — Git's immutable hashes and merge-commit default make the workflow a discipline you must remember and fear. Twigg deletes the fear by making the workflow the mechanism. Where Brethorst's guide teaches "your clone is disposable, rebase anyway," Twigg makes rebase the only way to submit.

**vs. [[gh-stack]].** gh-stack bolsters stacked PRs onto Git/GitHub with branch chains and a CLI. Twigg reaches the same destination by changing the substrate: stacks, versions, and review are native to the VCS, so there's no branch/base-chaining machinery to maintain. The trade is that gh-stack works with standard Git today; Twigg asks you to adopt a new VCS.

**vs. [[bucketvcs]].** bucketvcs re-implements Git's *storage* (object storage instead of block volumes) while speaking the standard Git protocol. Twigg re-implements the *entire VCS* — object model, IDs, wire protocol — and doesn't speak Git at all. Two opposite readings of "the future of source control": keep the protocol, change the storage; or change the protocol, keep boring local storage.

**vs. [[The End of Code Review]].** Twigg is the Critique-style human-review workflow — LGTM + OWNERS + threaded, version-anchored review — that the paper argues is collapsing under AI-generated volume. Twigg predates the agent wave and optimizes for small human-reviewable changes, which ironically is exactly the mitigation the paper reaches for from the other direction (small changes make review tractable).

---

*Sources: [[raw/monorepo]], [[summary/monorepo]]*
*Last updated: 2026-08-21*
