---
url: https://github.com/twigg-vc/monorepo
title: Twigg — open-source Critique (trunk-based code review VCS)
author: twigg-vc (Twigg)
date_fetched: 2026-08-21
date_published: 2026
---

# Twigg

Twigg is an open-source reimplementation of Google's internal code-review platform **Critique** — the workflow Google engineers use daily. It is explicitly **not a Git wrapper**: the entire version-control system is written from scratch in Go (~96K lines, 439 Go files across five binaries). AGPL-3.0 licensed.

Twigg is opinionated about one scenario: **collaboration in closed teams**. It bakes in trunk-based development and small stacked commits as the *default* rather than a discipline people must remember. Around that core it adds hierarchical code ownership (`OWNERS` files that cascade down directories), code review built for stacked changes that evolve version by version, and integrated CI/CD that runs on the trunk and triggers only on modified paths.

The monorepo contains five binaries plus a shared storage/VCS core:

- **`tw`** — the CLI (clone, commit, amend, rebase, submit, pull/push)
- **`twigg-web`** — the web server hosting repos, review, docs, and CLI binaries
- **`twigg-track`** — the job scheduler ("the server in which the runners run")
- **`twigg-runner`** — the job executor (Docker or LXD/VMs)
- **`twigg-vscode`** — a VSCode extension

Storage is a from-scratch engine shared by client and server: an append-only "datastrip" log for blob bytes with a SQLite index (`modernc.org/sqlite`, pure Go), delta-encoding consecutive blob versions. Commits are **versioned** (amend/rebase/submit produce a new version of the same logical commit, not a new commit), which is what makes "review version by version" possible.

The repo is a read-only GitHub mirror; actual development happens on Twigg's own hosted instance (dogfooding). The mirror snapshot is at Twigg commit `c/1341` (2026-08-12).
