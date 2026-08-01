---
url: https://github.com/rosenbjerg/local-review
title: "local-review"
author: Malthe Rosenbjerg
date_fetched: 2026-07-11
date_published: 2026-06
---

`local-review` is a local, single-user git review tool: review a branch's diff,
leave line/range comments, mark files reviewed, and export the review as
markdown for a coding agent. Go backend + React frontend, shipped as a single
binary with the frontend embedded via `go:embed`.

Comments survive rebasing and new commits through a two-tier anchoring system.
The primary tier uses `git diff <commit_sha> head -- path` with precise line
mapping — each comment records the commit SHA it was anchored against. A snippet
matching fallback handles worktree comments and edge cases. Staleness is
recomputed on every read, never trusted from persisted flags.

The backend is canonical; the frontend caches. SSE pings with `data: changed`
tell clients to refetch full state, avoiding desync and serialization
duplication. Reviewed files are tracked with SHA-256 content fingerprints —
changed files automatically lose their reviewed mark. Comments form two-level
threads (root + flat replies), and the markdown export uses backtick-safe
fencing that scans snippets for the longest backtick run to avoid prematurely
closing code blocks.

Architecture highlights: SQLite with `SetMaxOpenConns(1)` to make `foreign_keys`
pragma authoritative; custom diff parser that correctly handles `---`/`+++` in
content; idempotent schema migration via `PRAGMA table_info` checks; working
tree diff that includes untracked non-ignored files; and Shiki syntax
highlighting with JS regex engine to avoid WASM load failures.

The tool is designed for a specific workflow: a human reviews code, then hands a
markdown artifact to a coding agent to act on. No authentication, no multi-user
support — local-only on 127.0.0.1. This constraint simplifies every layer, from
the lack of a user table to the absence of permission checks.

Unlike AI code review tools ([[OpenCodeReview]], [[Metis — ARM AI Security Code
Review]]) which have AI *generate* reviews, local-review has humans write
reviews for AI to act on. Unlike workflow tools ([[Bram]]) it produces artifacts
without executing or approving anything. Unlike provenance tools ([[DeltaDB]])
it captures only the review moment, not the history of how code was produced.

---
*Source: [[raw/local-review]]*
*Last updated: 2026-08-01*
