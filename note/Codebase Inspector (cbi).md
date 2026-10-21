# Codebase Inspector (cbi)

`cbi` (joshuavial/codebase-inspector) is a deterministic codebase mapper that splits understanding of a repo into two halves: a tree-sitter-built structural model it derives itself, and a "judgement" half — summaries, names, concepts — that it outsources to AI agents through a validated task protocol. The tool contains no model calls and never runs the target's code; agents supply meaning, CI (`cbi check`) enforces that the meaning stays current. It is the inverse of most agent-context tools: instead of an agent building a mental model of the codebase each session and throwing it away, the codebase holds a persistent, incrementally-updated model that agents query.

---

## Architecture

Single Python package (`src/cbi/`, ~40 modules) installed via `uv tool install`; a thin Electron app (`app/`) shells the same CLI for humans.

- **Scan pipeline** (`scan` → `files.py`, `manifests.py`, `parse/`, `resolve.py`, `docs.py`, `tasks.py`): enumerate tracked files via `git ls-files` (submodules get pinned commits and drift flags), classify, detect deployables/packages from manifests (tsconfig outDir→rootDir source mapping, Vite entry points, package.json), parse changed files only, then resolve.
- **Storage** (`store.py`): one SQLite file per repo at `.cbi/model.db` — `nodes`, `edges` (typed + `source` + confidence 0–1), `diagnostics`, `file_state` for incremental scans, `coverage_lines`/`results` for ingested artifacts, a `tasks` table, and an FTS5 table over names/signatures/summaries for `cbi search`. Schema version lives in `user_version`; mismatch = drop and rebuild, because the model is derived.
- **Node IDs** (`ids.py`): stable across machines. Workspace ID is the normalised git remote (`github.com/owner/repo`, credentials stripped, no commit); symbols are `<file id>#<qualified name>`, with sha1-signature suffixes for duplicate definitions. This is what makes model diffing (`cbi diff`) possible.
- **Viewer** (`build.py` + `viewer/`): plain JS, no build step, opens from `file://`; the only vendored library is MiniSearch. The Electron app is explicitly a thin shell over the installed CLI.
- **Desktop**: `app/` TypeScript files (`project.ts`, `view-state.ts`, `compare.ts`) mostly wrap CLI output; well-tested (~20 test files with `.test.ts` siblings).

## Key techniques

- **The judgement task protocol** (`tasks.py`) is the core novelty. Task kinds: `summarise-files` (batches of 15, but cached *per file* by content hash), `summarise-group` (bottom-up, blocked until children are done), `confirm-structure` (agent corrects detected deployables; acceptance writes `.cbi/structure.json` which is re-applied on every scan), `summarise-deployable`, `summarise-workspace`. Answers must echo the brief's `input_hash` or submit rejects them — staleness is handled with the same optimistic-concurrency pattern a distributed system would use. Answers go to a shared cross-repo cache (`~/.cache/codebase-inspector/answers.db`, `ATTACH`ed to the model connection so submit is one transaction); identical inputs are only ever judged once, across all repos.
- **Confidence-scored edge resolution** (`resolve.py`): imports followed through re-exports to the *defining* file; calls bound only when the callee can bind (same-file, imported-by-name, `self.`/`this.`), with local shadowing checked; tests linked by call (1.0), literal read (1.0), name match (0.8), import (0.5). Ambiguous calls get *no* edge plus a diagnostic — an honest "I don't know" instead of a plausible wrong answer.
- **Facts table for cheap re-resolution**: parsers store each file's imports/exports/calls as JSON keyed by file ID, so resolution reruns over all files without reparsing unchanged ones.
- **Deterministic CI gate** (`check.py`): scans, then checks that files changed since the merge base have summaries and (in team mode) committed `answers.jsonl`; also regex-scans answer text for high-entropy tokens and known key prefixes (AKIA, `ghp_`, `sk_live_`…) so agents can't paste secrets into committed summaries. Nothing fetched, no model, no network.
- **`cbi prime` as agent bootstrap**: prints one of three ≤60-line guides (not initialised / tasks pending / complete) with examples on the *current repo's real node names*. Help text is treated as the agent's documentation — hand-written for every subcommand.
- **Artifact ingest** (`ingest.py`): detects lcov / coverage.py JSON / JUnit XML by content; coverage.py `contexts` produce `tests` edges at confidence 1.0 because the test *executed* those lines. The docs are refreshingly honest that compiled-output coverage must be source-mapped before ingest, because "path rewriting alone is wrong — compilation moves lines."

## Design decisions

- **No LLM in the tool** is the organising constraint, and it buys determinism (CI-safe, reproducible models, diffable across commits) at the cost of a clunkier protocol: agents must round-trip through `tasks`/`task`/`submit` rather than the tool "just knowing". The bet is that the structural half is where determinism pays and the semantic half is where agents are cheap and good.
- **Never executes target code, never installs its dependencies** — which is exactly why SCIP indexers were rejected: they need the repo's environment. The trade-off is coarser call resolution than a compiler-grade indexer; the project accepts ambiguity and records it as diagnostics rather than guessing.
- **tree-sitter pinned below 0.26** because 0.26.0 segfaults on node access after a few parses — the kind of version-floor decision usually only visible in commit messages, here documented in the ADR register.
- **Containment as `parent_id`, not edges** keeps the edge table purely about relationships; derivation (derived model) is rigorously separated from judgement (committed answers), which is what allows `cbi check` to rebuild one and verify the other.

## Comparison notes

- Compared to [[Maguyva]], which offers agents a pre-built graph-ranked codebase map over MCP, cbi moves the division of labour one level down: Maguyva serves structure, cbi also brokers *semantic* content (summaries, concepts) as agent-supplied, cache-stamped data — closer to a protocol than a tool.
- It is a direct answer to [[The Knowledge Chipper]]'s complaint that agents burn tokens rebuilding a mental model of the codebase every session and that PRs arrive as parcels of undocumented complexity: cbi makes that model a committed artifact that survives the session and the PR.
- Against [[Context Graphs]]' argument that agent memory should be typed, structured graph edges rather than vector similarity: cbi agrees structurally (typed edges with confidence and source in SQLite), but its nodes are code entities derived by parsing, not decision records captured on the write path.
- It also supplies a concrete primitive for [[Towards Self-Driving Codebases]]' "agent-legible environments" argument — `cbi prime` plus the query commands are precisely an environment component built for agents first, humans second.
- Related in spirit: [[DeepWiki]] (generated docs for repos), [[Wuphf — Karpathy-Style Agent Wiki]] (agent-maintained wiki), and [[Mindwalk]] (visual replay of agent sessions over a codebase map).

---

*Sources: [[raw/codebase-inspector]], [[summary/codebase-inspector]]*
*Last updated: 2026-10-21*
