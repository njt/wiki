# Greenroom (jesserobbins)

greenroom is a Claude Code skill (and generic agent skill, distributable via `npx skills add`) that sets up and maintains a "green room" beside every public repo: a private, git-versioned notes repo holding the design docs, PRDs, drafts, and PR-review thinking that generate rich context for agents but that nobody wants to publish. The theatre metaphor is the point — the public repo is the stage, and you only step on it when prepared. What makes it more than a mkdir script is the discipline around it: leak-hygiene rules written into the agent-facing instructions, defensive scaffolding that refuses dangerous directories, and a recovery tool (`collect`) for docs already stranded in public history.

---

## Architecture

The entire installable payload is one skill directory. The code is a single stdlib-only Python script, `skills/greenroom/scripts/greenroom.py` (1,839 lines), with four subcommands:

- **`retrofit <path>`** — wraps an existing repo. Moves it into a new wrapper folder as `<name>-public/`, scaffolds `<name>-private/`, writes all orientation files. Handles the in-place collision case (repo already sits at the would-be wrapper path) by moving to a temp path, creating the wrapper, and moving it back in — with crash-safe restore so a failed move never strands the repo.
- **`new <name>`** — fresh project, optionally `--clone <url>` or `--init-public`.
- **`sync`** — re-scans the wrapper for git repos and tops up everything: workspace roots, sibling access grants, README repo map. This is the re-run trigger after dropping a new fork under the wrapper.
- **`collect`** — recovers private-shaped docs from public git history into the private repo.

Around the script sit deliberately hollow slash commands (`commands/add.md` etc. are 8–10 lines that just invoke the skill), reference docs (`references/layout.md`, `operations.md`, `workspace.md`, `collect.md`), and a 2,202-line black-box smoke test (`tests/smoke.sh`) that builds throwaway repos in a temp dir and asserts on the reliability-critical paths — sibling repos in the parent, crash-safe restore, component-boundary matching in config checks.

The generated layout is the real artifact: a non-git wrapper folder holding sibling git repos, with a `.greenroom` identity marker (content is only `{"schema": 1}` — everything else is deliberately *not* stored because it's derivable and would go stale).

## Key techniques

**AGENTS.md as the universal adapter layer.** greenroom standardizes on AGENTS.md (read natively by 25+ agents) and writes exactly two bridges: a `CLAUDE.md` containing the single line `@AGENTS.md` (Claude Code's `@`-import), and `.gemini/settings.json` with `{"context": {"fileName": "AGENTS.md"}}`. Everything else is "documented, not configured." Per-repo pointers are written only where an `AGENTS.md` already exists.

**Wrapper-as-launch-home.** The one rule — `cd <wrapper> && <agent>` — solves three problems at once: all repos are under cwd (no `--add-dir` wiring needed), session history stays in one bucket instead of fragmenting across `-public`/`-private`, and each child's AGENTS.md loads lazily on first touch. The VS Code workspace reinforces it: `terminal.integrated.cwd` points at the wrapper (parent of the canonical repo), and a "Claude Code (canonical)" task launches there.

**Defense-in-depth access.** The workspace file grants nothing (it's for VS Code). For a stray agent launch *inside* one repo, every repo gets a git-excluded `.claude/settings.local.json` listing its siblings under `permissions.additionalDirectories` as `../<name>` — siblings, never `..`, which would over-grant the whole parent. The file is also added to `.git/info/exclude` so the private paths never appear in `git status` or reach the public repo.

**A leak-hygiene doctrine, not just a layout.** The templates instruct the agent: reference public artifacts by GitHub URL (commit SHA or PR number), *never by local path* — "the path itself is a small leak." A one-time global gitignore (`~/.config/git/ignore`) covers `.notes`, `NOTES.md`, `SCRATCH.md`, `*.private.md`, `.private/` so those names can never land in a public repo from any clone.

**Wrapper classification is deliberately paranoid.** `_is_project_wrapper` requires a non-git directory holding git repos *plus* a positive signal: a schema-validated `.greenroom` marker, a workspace file carrying greenroom's `greenroom: {wrapper: true}` sentinel, or a `*-private` sibling. A stray `.greenroom` from another tool, a non-JSON marker, or a generic clone parent like `~/GitHub` all fail the check — because misclassification would mean granting `..` over the home directory.

**The forbidden-root floor.** `$HOME`, the filesystem root, and standard directories (`Documents`, `Desktop`, `.config`, ...) are never wrappers or scaffold targets regardless of signal — motivated by a real incident where a stray `.code-workspace` made `$HOME` classify as a wrapper and greenroom scaffolded `~/CLAUDE.md` into nearly every session. The optional `GREENROOM_ROOT` env var is a boundary, not a target: greenroom works *under* it, never *at* it.

**JSONC tolerance with a hard line.** `write_code_workspace` merges into existing `.code-workspace` files using a hand-rolled trailing-comma stripper (stdlib `json.loads` rejects what VS Code itself writes), but refuses files with real `//` comments — leaving them untouched rather than risk clobbering user edits. Own-authored tasks (label-prefixed `Claude Code (`) are refreshed in place; user tasks are preserved.

**Legacy-template migration.** `migrate_claude_to_agents` recognizes greenroom-authored `CLAUDE.md`s from *older releases* by matching against every superseded template snapshot in `templates/legacy/`. The changelog documents why the legacy generations are load-bearing: matching only the current template silently reclassified old wrappers as "hand-edited." A drift guard in the smoke test walks the template's git history and fails if a shipped generation is missing.

## Design decisions

The big one is **two repos, not one**. The design notes (`docs/design.md`) dispatch the alternatives: a gitignored `private/` dir is one `git add -f` from public and isn't versioned anyway; a long-lived private branch shares origin and history (one `git push --all` publishes everything); one central private repo for all projects loses per-project locality. Separate repos mean separate `.git/`, separate remotes, separate visibility — git doesn't know the sibling exists.

**The script never pushes, never commits in the public repo, never creates a remote.** It prints `gh repo create --private` commands for the user to run on explicit confirmation — and the skill instructions tell the agent to relay them verbatim and never suggest a public variant. Same philosophy in `collect`: copy-only, dry-run by default, never rewrites public history (scrubbing with `git filter-repo` has real blast radius and is deliberately out of scope).

**Directory naming as tool-compat**: `<project>-private/`, not `private/`, because tooling infers project identity from basenames — remotes, IDE labels, session bucketing. A dozen folders all named `private` would collapse together.

Trade-offs: everything is idempotent and additive (re-runs top up, never overwrite — even hand-edited files are detected and preserved), which costs complexity in the template-matching machinery. And the whole thing is a *convention* enforced by generated prose, not a technical barrier — a misbehaving agent in the public repo can still write where it shouldn't. The defense is layered instructions plus git hygiene, not hard enforcement.

## Comparison notes

greenroom is the infrastructure complement to [[Superpowers for Humans (O'Reilly)]] — it scaffolds a home for exactly the "knowing what you want" documents (PRDs, designs, plans) that Vincent's management-first philosophy says are the scarce human contribution, and it's explicitly built as a companion to obra/Superpowers. Where [[Agent Skills Library (dzhng)]] shows the skill format as a *workflow engine* (plan-slice-verify loops), greenroom shows it as *environment engineering*: the skill's job is to reshape the filesystem so the agent's defaults are safe. Its AGENTS.md-everywhere adapter strategy is the write-up counterpart to [[Benchmarking AGENTS.md Changes]]'s measurement angle, and its launch-at-the-wrapper session-bucketing is a small but concrete answer to the context-continuity concerns in [[Maybe Coding Agents Don't Need a Bigger Memory]] — a per-project layout that keeps a session's files, history, and orientation in one place rather than a memory architecture.

Tags: #tool #project #agents #claude-code #git #workflow

---
*Sources: [[raw/greenroom]], [[summary/greenroom]]*
*Last updated: 2026-11-02*
