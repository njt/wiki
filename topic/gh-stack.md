# gh-stack — GitHub Stacked PRs CLI

GitHub's official CLI extension for managing stacked branches and pull requests. Rather than submitting one massive PR, developers split work into an ordered chain where each branch builds on the one below it; reviewers see only the diff for their layer. The tool automates branch creation, cascading rebases, PR base-branch chaining, push coordination, and batch merging — the mechanical chores that make stacked PRs painful to do by hand. Built in Go (~52K lines) as a `gh` extension, with interactive Bubble Tea TUIs and a local-first state model.

#tool #project #git #cli #code-review

---

## Architecture

The project is a **single monolithic `gh` extension** — `main.go` (10 lines) delegates to `cmd/root.go` which constructs a Cobra command tree. No subprocess fan-out or plugin system.

### Package layout

```
main.go                          — entry point
cmd/                             — CLI commands (Cobra), one file per command
internal/
  config/config.go               — shared Config: terminal, colors, client injection
  github/                        — GitHub API client (GraphQL + REST)
    github.go                     — PullRequest, Client, RemoteStack, merge ops
    client_interface.go           — ClientOps interface (28 methods, mockable)
    mock_client.go                — test mock
    merge_async.go                — async merge orchestration
  git/                           — Git operations abstraction
    gitops.go                     — Ops interface (~80 methods), defaultOps implementation
    git.go                        — low-level git execution (run, runSilent, runRebaseCommand)
    mock_ops.go                   — test mock
  stack/                         — Core data model and persistence
    stack.go                      — Stack/BranchRef/PullRequestRef types, Load/Save, staleness checks
    lock.go                       — file locking (flock on Unix, LockFileEx on Windows)
    lock_unix.go, lock_windows.go — platform-specific lock implementations
    schema.json                   — JSON Schema for .git/gh-stack
  modify/                        — Stack restructuring engine (apply plan, preconditions)
  tui/                           — Bubble Tea terminal UIs
    submitview/                   — PR submission editor (title/description/draft, Ctrl+S to submit)
    modifyview/                   — stack restructuring TUI (drop, fold, insert, reorder, rename)
    checkoutview/                 — interactive stack listing picker
    mergeview/                    — merge wizard
    stackview/                    — stack display (used by view and as sub-component)
    shared/                       — rendering primitives, theme, header, scroll, logo
  theme/                         — terminal theme detection (auto/light/dark)
  branch/                        — branch name utilities
  pr/                            — PR template engine
skills/gh-stack/                 — AI agent skill: SKILL.md + references/ for coding agents
docs/                            — Astro documentation site
```

### Core data model (`internal/stack/stack.go`)

```go
type Stack struct {
    ID       string      `json:"id,omitempty"`
    Number   int         `json:"number,omitempty"`
    Trunk    BranchRef   `json:"trunk"`
    Branches []BranchRef `json:"branches"`  // ordered bottom-to-top
}

type BranchRef struct {
    Branch      string          `json:"branch"`
    Head        string          `json:"head,omitempty"`    // trunk HEAD SHA
    Base        string          `json:"base,omitempty"`    // parent HEAD at last sync
    PullRequest *PullRequestRef `json:"pullRequest,omitempty"`
    Queued      bool            `json:"-"`                 // transient, from API
}
```

The `Base` field on stacked branches stores the parent branch's HEAD SHA at the time of last sync/rebase. This is the key invariant that makes `needsRebase` detection efficient and `--onto` rebase (for merged parents) possible.

### Persistence strategy

Stack metadata is stored in `.git/gh-stack` as JSON with schema versioning. This file is **local-only, never committed**. The design has three layers of safety:

1. **Optimistic concurrency** (`stack.go:395–428`): `Load()` computes a SHA-256 of the raw bytes. `Save()` re-reads the file under lock and compares checksums — if they differ, it returns a `*StaleError` to prevent lost updates from concurrent `gh stack` processes.
2. **Advisory file lock** (`lock.go`): `flock` on Unix, `LockFileEx` on Windows. Lock timeout is 5 seconds with 100ms retry intervals. The lock file (`gh-stack.lock`) is intentionally left on disk after unlock to prevent the inode-reuse race.
3. **Non-blocking variant** (`SaveNonBlocking`): Best-effort save used during view/status operations. Silently skips if the lock is held or the file was modified — avoids blocking user-facing commands.

### Remote integration

Two API surfaces (`internal/github/github.go`):
- **GraphQL** (`go-gh` + `shurcooL-graphql`): Fast queries for PR lookup, creation, status. Allows single-request queries for "find open PR by branch name" with merge queue and auto-merge state.
- **REST** (`go-gh`): Stack CRUD via `/repos/{owner}/{repo}/stacks` endpoints, PR base updates via `PATCH`, merge operations.

The `ClientOps` interface (`client_interface.go`: 28 methods) enables complete testability — commands receive an interface, tests inject a `MockClient`.

### TUI architecture

All interactive screens use Charm's [Bubble Tea](https://github.com/charmbracelet/bubbletea) framework. Each view is a self-contained package:

- **`submitview`** — The most complex TUI (~800 lines model + ~500 render). Two-panel layout: left panel shows branch timeline with checkboxes for inclusion; right panel edits title/description/draft for the focused branch. Supports mouse events (with a leaky-mouse-key filter for terminal escape sequences that split across reads). Uses `tea.ExecProcess` for `$EDITOR` launch (with `tea.EnableMouseCellMotion` re-arming on return).
- **`modifyview`** — Stack restructuring with operations mapped to vi-style keys: `x`=drop, `d`=fold-down, `u`=fold-up, `i`/`I`=insert below/above, `r`=rename, `Shift+arrows`=reorder, `z`=undo, `Ctrl+S`=apply all.
- **`checkoutview`** — Searchable picker listing all local and remote stacks, with tabs (All/Local/Remote) and status bars.
- **`mergeview`** — Wizard for selecting merge target and method.

Shared primitives in `tui/shared/` handle rendering, theming, scroll management, header chrome, and the invertocat logo.

### Git abstraction

The `git.Ops` interface (`gitops.go`) exposes ~80 git operations behind a mockable facade. The production implementation (`defaultOps`) shells out to the real `git` binary, but the package uses a replaceable global: `var ops Ops = &defaultOps{}` with `SetOps()` for test injection. This is arguably a weakness — global mutable state is fragile — but it avoids threading an ops parameter through every command. The MockOps implementation tracks calls and returns configurable values, enabling tests like "verify push was called with force-with-lease for exactly these branches."

---

## Key Techniques

### Cascading rebase with conflict recovery

The rebase engine (`cmd/rebase.go` and shared helpers in `cmd/utils.go`) is the project's most complex algorithm. It rebases branches in order from trunk upward, with three notable features:

1. **`--onto` mode for merged parents**: When a branch's parent PR has been merged, the rebase automatically switches to `git rebase --onto <trunk> <old-parent-SHA> <branch>`, correctly replaying only the branch's own commits on top of trunk. The `Base` field on each `BranchRef` (saved at last sync) provides the `<old-parent-SHA>`.
2. **JSON-persisted rebase state** (`cmd/rebase.go:29–43`): On conflict, the full state (branch indices, remaining branches, original refs, onto mode, options) is serialized to `.git/gh-stack-rebase-state`. `--continue` reloads, verifies stack hasn't been modified since, and resumes. `--abort` restores all branches to their pre-rebase SHAs via `git reset --hard`.
3. **Safety nets**: Before rebasing, all branch HEADs are captured. On sync failure (non-interactive), the rebase is aborted and every branch restored. On verify failure (post-rebase ancestry check), the same restore path runs.

### Force-with-lease the right way

The push implementation (`git/push.go`) builds explicit per-branch `--force-with-lease` arguments by resolving each branch's tracking ref SHA. Branches with no tracking ref get an empty expected value (`--force-with-lease=refs/heads/<name>:`), meaning "must not exist." This is correct where many tools get it wrong — they pass the local branch name and let git negotiate from upstream config, which fails when tracking refs are stale or missing.

### Remote-ahead reconciliation

`sync` handles the case where the GitHub stack has PRs the local stack doesn't (someone added a PR via the web UI or another machine). The reconcile step (`cmd/sync.go:118–137`) compares local and remote stacks:

- If the remote is a **clean superset** (all local PRs exist in the same order on the remote, plus more on top), branches are fetched and appended automatically — no prompt.
- If the stacks have **diverged** (both sides have unique PRs), an interactive prompt offers three choices: use remote as source of truth, delete the remote stack, or cancel. In non-interactive mode, divergence aborts safely without pushing anything.
- If the divergence resolution changes the stack, the user's checkout is preserved or moved to the nearest surviving branch.

### All-or-nothing batch merge

`gh stack merge <target>` merges the target PR **plus every unmerged PR below it** in a single operation. If any one can't merge, none do. The command delegates to an async merge orchestration (`internal/github/merge_async.go`) that polls for completion. When the base branch uses a merge queue, the stack is added to the queue instead, and the method-selection flag is silently ignored with a warning (the queue picks the method).

### `gh stack link` — zero-local-state stacking

The `link` command (`cmd/link.go`) creates or updates a stack on GitHub **without touching local tracking state**. It takes branch names or PR numbers in stack order, pushes branches, creates PRs, sets correct base branches, and creates the stack — all in one command. This is designed for users who manage branches with other tools (jj, Sapling, git-town) and only want the GitHub Stack object. The command is purely additive: existing PRs are never removed from a stack.

### AI agent skill

The `skills/gh-stack/SKILL.md` file (182 lines) is a structured prompt for coding agents, telling them how to use `gh stack`. It covers setup, non-interactive mode (with a critical table: "always run `view --json` / never run bare `view`"), branch placement rules, the core loop, sync/merge procedures, JSON output schema, exit codes with recovery instructions, and constraints ("stacks are strictly linear, no non-interactive reorder"). Three reference files cover stack design, command details, and troubleshooting. This is a practical implementation of [[10 Principles for Agent-Native CLIs]] — the CLI already supports agents (`--json`, `--auto`, `--yes`, structured exit codes), and the skill file is the missing documentation layer.

---

## Design Decisions

### Optimized for: safe automation and interactive use

The tool makes different trade-offs than the obvious approach of "just write a bash script around `git rebase`":

| Choice | What they optimized for | What they sacrificed |
|--------|------------------------|---------------------|
| Local-first JSON state | Speed, offline operation, no API dependency for basic ops | Risk of local/remote divergence (mitigated by `sync` reconciliation) |
| CSV-style exit codes (0–10) | Scriptability by human and AI consumers | Having to document/maintain a taxonomy |
| Interactive TUIs with `--auto`/`--yes` escapes | Rich UX for humans, clean CLI for machines | Code complexity doubling (every command has two UX paths) |
| Optimistic concurrency + file locks | Correctness under concurrent use | Lock contention risk (5s timeout) |
| Shelling out to `git` binary | Simplicity, correctness (git is the authority on git operations) | Performance overhead per-operation, platform dependency |
| GraphQL for PR queries, REST for stack CRUD | Fast targeted queries + REST where GraphQL schemas don't exist | Maintaining two API integration patterns |
| Per-command `Config` injection | Testability (no global state for config) | Every command must thread `cfg` through |

### The locking approach

Using an advisory file lock with a timeout, rather than a lock-free data structure or a dedicated daemon, is pragmatic. The lock window is milliseconds (only during file write), so contention is rare. The checksum-based staleness check is the real safety net — the lock just narrows the race window to zero.

One subtle design note: the lock file is **never deleted**, only unlocked. This prevents the classic inode-reuse race where Process A blocks on a lock, Process B deletes the file and creates a new one, and Process A wakes up holding a lock on the old (now untracked) inode. Leaving the file on disk means everyone locks the same inode.

### GraphQL + REST dual surface

This is a constraint, not a choice — the GitHub Stacks API is REST-only, but PR queries are far more efficient via GraphQL (single query for PR-by-branch with merge queue state). The code keeps both clients cleanly separated: GraphQL for reads, REST for writes. The `ClientOps` interface abstracts this from callers.

### Comparison to alternatives

**vs. git-town**: git-town is a more general git workflow tool (GitHub, GitLab, Bitbucket) with a sync-first philosophy. gh-stack is narrower — it only does stacked PRs on GitHub — but goes deeper on the UX (interactive TUIs, merge orchestration, remote stack reconciliation). git-town has no AI agent skill.

**vs. Graphite / Sapling**: These are full-stack solutions that replace parts of the git workflow. gh-stack is a `gh` extension that works with standard git — the stack file is the only addition. It doesn't require a new VCS or server-side component. The trade-off is that Sapling/Graphite handle more complex DAG topologies (non-linear histories) while gh-stack enforces strict linear stacking.

**vs. manual `git rebase --onto`**: The baseline approach works for one person who knows git well. The value-add is in the mechanical correctness (correct `--force-with-lease`, merged-parent `--onto` detection, conflict state management) and the collaborative features (remote stack reconciliation, batch merge, review-aware sync). The TUI layers are pure UX — they don't enable new capabilities, but they make existing ones discoverable and safe.

---

## Relevance to AI Agents

gh-stack's AI agent integration is a notable design choice: the tool ships with a structured skill file for coding agents, not just human documentation. This reflects a growing consensus that CLI tools should be **agent-native by design** — providing JSON output, structured exit codes, and non-interactive modes as first-class features, not afterthoughts.

The `gh stack sync` command's remote-ahead auto-reconciliation is particularly agent-friendly: when PRs have been added to a stack on GitHub (by a human via the web UI, or by another agent instance), sync pulls them down automatically without prompting. The only prompt trigger is genuine divergence, which aborts cleanly in non-interactive mode rather than making an unsafe guess.

Stacked PRs as a workflow also address a key pain point in agentic development: agents produce large diffs, and stacked PRs break those into reviewable layers. This aligns with the findings in [[Agentic Code Review]] — the bottleneck isn't writing code, it's trusting it. Smaller, layered PRs make verification tractable.

The ultimate version of this idea is [[Twigg]], which reaches the same destination by replacing Git rather than extending it: stacked commits and versioned amends are native to the VCS itself, so the branch-chaining and base-tracking machinery gh-stack exists to manage simply doesn't exist there.

---

*Sources: [[raw/gh-stack]], [[summary/gh-stack]]*
*Last updated: 2026-08-07*
