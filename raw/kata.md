---
url: https://github.com/wesm/kata
date_fetched: 2026-07-05
backfilled: true
---

The issue tracker built for coding agents and the humans steering them.

Coding agents need somewhere durable to track work: not a chat thread, not a markdown to-do list. kata gives them a local task ledger they can drive from the CLI: create, claim, relate, and close issues with evidence. Humans supervise the same work in a terminal UI. By default, issue state lives in a local SQLite database, so your repo stays clean and no hosted tracker is required. When a team of humans and agents needs to share, you can opt into a remote daemon or federation.

The documentation in `docs/` is the definitive guide, published with
Zensical at https://katatracker.com/.


Pre-1.0:kata publishes versioned pre-1.0 releases. The CLI, daemon, and TUI are usable, but command contracts and UI details can still change before a stable release.

macOS or Linux:

`curl -fsSL https://katatracker.com/install.sh | bash`Windows PowerShell:

`powershell -ExecutionPolicy ByPass -c "irm https://katatracker.com/install.ps1 | iex"`The installer detects your OS and CPU architecture, downloads the latest GitHub
release archive, and verifies it against `SHA256SUMS` before installing. Confirm
the install with:

`kata version`Release builds update themselves with `kata update`. Linux `.deb` and `.rpm`
packages are published for `amd64` and `arm64`. Prefer to build from source?
kata needs **Go 1.26 or later**:

`go install go.kenn.io/kata/cmd/kata@latest`Go installs to `$(go env GOBIN)`, falling back to `$(go env GOPATH)/bin` (often
`~/go/bin`); put that directory on your `PATH`. See
Install for package downloads, manual release
downloads, and build-from-source steps.

```
cd your-repo
kata init                              # bind this workspace to a kata project
kata create "fix login race"           # prints a short id, e.g. abc4
kata list                              # see open work
kata show abc4                         # inspect by short id
kata tui                               # browse and triage interactively
```
`kata create` prints each issue's short id; use it in later commands. Close only
when the work is complete and verified:

```
kata close abc4 --done \
  --message "Fixed the login race and verified the relevant tests pass." \
  --commit <sha>
```
For agent-heavy workspaces, `kata init --with-agents` also writes a managed kata
briefing into agent guidance files. It refreshes existing real `AGENTS.md` and
`CLAUDE.md` files, or creates `AGENTS.md` when neither exists, without
overwriting the rest of either file.

- **Built for agents.**Stable short refs,- `--json`and- `--agent`output, idempotent creates, semantic-aware search, a claim flow, and predictable failure modes agents can script against.
- **Made for humans too.**- `kata tui`browses, triages, and supervises agent-written work over the same data. No raw JSON required.
- **Local-first, repo-clean.**One Go binary, no runtime dependencies. Issue state lives in SQLite under- `KATA_HOME`; your repo commits only a small, secret-free- `.kata.toml`.
- **Auditable by design.**Closing an issue is an explicit completion claim with a reason, message, evidence, and actor attribution, on top of editable comments and durable events.

kata is intentionally small. It is not a project-management suite, a git workflow engine, or an agent worker pool. It is a durable task ledger that humans and agents can both operate.

It is also not a SaaS issue tracker. Linear, Jira, GitHub Issues, ClickUp, and similar tools are shared online systems for planning, dashboards, assignment, and cross-team reporting. kata is local-first, instant from the CLI/TUI, and designed around agent-first ergonomics: stable refs, predictable output, idempotent creates, claim flows, and evidence-based closes. See Comparisons with SaaS issue trackers for the matrix.

Beads keeps issue state in a
project-local `.beads/` Dolt database with native history, branching, and
push/pull. git-bug stores issues as git
objects under custom refs and syncs them over `git push` and `git pull`. kata
makes a different bet: the ledger is a local service next to your workspaces,
not data carried in the repository. That keeps the workspace clean, works the
same in non-git directories, and keeps issue history out of code history. The
trade-off is that kata does not ride git remotes for sharing; the remote daemon
and federation cover that instead.

Moving from Beads? See
Migrating from Beads.
`kata import --source-format beads` drives the `bd` CLI and merges your issues
into a kata project.

The docs site is the definitive reference:

- Get started: Quickstart · Install · Changelog
- Guide: Concepts · Workspaces and projects · Semantic search · Migrating from Beads
- Reference: CLI · Configuration
- Workflows: Agent workflows · Sharing models
- Operations: GitHub sync · Remote daemon · Federation · Hosted mode · Backup and restore

Run `kata quickstart` (alias `kata agent-instructions`) for the operating
contract: search before creating, pass an idempotency key on create, prefer
`--agent` output, claim work with `kata claim`, and close only when the work is
verified. Close each verified issue promptly with valid evidence and a
substantive message. Agent workflows is the same
contract in long form.

See Contributing for the repository layout
and local checks (`make test`, `make lint`, `make vet`, `make nilaway`).
Licensed under the terms in LICENSE.
