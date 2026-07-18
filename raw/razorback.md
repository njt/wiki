---
url: https://github.com/spacedock-dev/razorback
title: Razorback
author: CL Kao (clkao@datarecce.io)
date_fetched: 2026-07-18
date_published: 2025-06
---

# Razorback

A Python CLI (`rk`) built on Harbor for reproducible agentic benchmark research. It turns benchmark runs into defensible numbers: freeze the spec, run the job, score the result, audit for leakage, and inspect run-dir artifacts.

## Architecture

Razorback is a Python monorepo (~14,500 lines of Python source) using Typer for CLI, Pydantic for schema validation, Harbor 0.6.6 as the benchmark runner framework, and uv as the package manager. The project is authored by CL Kao of DataRecce and bears the same author's fingerprints as Spacedock (the multi-agent orchestration system).

### Layer Map

```
rk CLI (Typer)
  ├── rk freeze       → provenance/freeze_cmd.py → spec/freeze.py
  ├── rk run          → cli/run.py → translate.py → Harbor Job.create()
  ├── rk score        → score/render.py ← runs/aggregate.py (filesystem-driven)
  ├── rk audit        → audit/cli.py → audit/taint.py (trace scan)
  ├── rk runs diff    → diff/diff.py (paired comparison)
  ├── rk research     → cli/research.py (scaffold generator)
  ├── rk baseline     → cli/baseline.py (baseline management)
  └── rk registry     → cli/registry.py (plugin management)
```

### Core subsystems

1. **Spec system** (`src/razorback/spec/`) — 343-line Pydantic schema with `extra="forbid"` on every model. Discriminated unions for agent blocks (nop, claude-cli, codex, spacedock_solver) and benchmark blocks (local, harbor, harbor-local). Specs carry experiment identity, agent configuration, benchmark reference, trial count, concurrency, observers, provenance preferences, and experiment metadata.

2. **Translate** (`src/razorback/translate.py`) — 656-line bridge between Razorback specs and Harbor JobConfig. Key mappings:
   - Agent blocks → `AgentConfig(import_path=...)` referencing Razorback's runtime adapters
   - Benchmark blocks → task paths via Harbor dataset resolution or plugin invocation
   - Auth resolution via `.env` files (Anthropic API key or Claude OAuth token)
   - Environment config with proxy blocking (`http_proxy=""`, etc.) and freeze-dir bind mounts

3. **Agent system** — Three-layer hierarchy:
   - **Runtime adapters** (`_runtime/claude.py`, `_runtime/codex.py`): Subclass Harbor's installed agents (ClaudeCode, Codex) with Razorback policy. Claude adapter adds tool allow/deny policy and plugin staging. Codex adapter disables web search, installs public-lookup guards (shell wrappers that block curl/wget/npm/pip), clears proxy env during install, and rejects unsupported kwargs.
   - **SpacedockSolverAgent** (`spacedock_solver.py`): Outer orchestration wrapper (609 lines). Computes sealed identity hash, manages content-addressed freeze/resume tree via git, runs workspace preflight, builds inner runtime adapter, and injects a first-officer prompt that dispatches `spacedock:ensign` workers.
   - **Seal** (`seal.py`): Cryptographic identity that pins what an experiment run IS. Six inputs → one hex hash: model + sampling + solver_workflow_content_hash + prompt_content_hashes + spacedock_skill_version + harbor_agent_kwargs + optional task_identity. Canonical JSON encoding (sorted keys, separators `(",",":")`), first 32 hex chars of SHA-256.

4. **Provenance/freeze** (`provenance/`): Resolves dynamic inputs into pinned values:
   - Model version (Anthropic API: `client.models.retrieve(model_alias)`)
   - Docker image digest (`docker image inspect --format '{{ .Id }}'`)
   - Agent CLI binary hash (SHA-256 of the binary on $PATH)
   - Harbor version, git SHA, plugin inventory (entry points scan)
   - Solver workflow content hash (recursive directory hash, frame-level boundary encoding)
   - Writes `spec.frozen.yaml` + `provenance.yaml`

5. **Budget** (`budget.py`): Running-cost gate. A JSON file with exclusive-lock atomic writes (tempfile+rename, fsync'd) that tracks experiment spend across multiple invocations. Pre-launch: `decide_budget()` compares `current_total_usd + estimate_usd > max_budget_usd`. Post-run: `stamp_completed()` updates the invocation record. Handles subscription-auth null-cost gracefully (cost_known: false → use estimate as proxy).

6. **Score** (`score/`): Per-query stratified pass@1. Each (dataset, query_id) cell is a binomial proportion with Wilson CI at the cell level only. Dataset stratum is the mean of per-query proportions — not binomial, so stratum CI is always null. Against-constant comparison: matches/above/below point verdicts.

7. **Audit/taint** (`audit/`): Post-hoc trajectory scan ported from dataagentbench's taint scanner. Three scan categories:
   - **forbidden_lookup**: curl/wget/npm install/pip install of canonical-data libs (datasets, huggingface, transformers, evaluate)/huggingface-cli/web_search/web.run
   - **trace_coverage**: missing or partial subagent dispatch traces, hook reconciliation failures
   - **attempt_incomplete**: timed-out or non-terminal worker status (frontmatter status != "done")

8. **Runs aggregate** (`runs/aggregate.py`): Filesystem-driven post-harbor aggregator. Reads trial directories (discovered by checking for result.json), resolves strata from view_manifest.json (preferred) or trial directory name parsing (fallback), handles DAB batch mode fanout (one trial → N per-query rows via reward_per_query.json), and writes manifest.json, summary.json, per_trial_outcomes.json, and events.jsonl.

9. **Diff** (`diff/diff.py`): Paired comparison between two run-dirs. Exact-McNemar per query, Wilson CIs per arm, paired bootstrap CI on stratified delta, and power analysis (MDE at fixed N).

### Plugin architecture

The DAB plugin (`packages/razorback-plugin-dab/`) demonstrates the plugin model:
- Discovered via `razorback.plugin_args` entry point
- Generates harbor task directories (task.toml + instruction.md + tests/)
- Supports 12 DataAgentBench datasets with docker-compose sidecar databases (Postgres, Mongo)
- Three workspace variants: direct-minimal, direct-structured, spacedock
- Query modes: batch (one task per dataset) or per-query (one per question)
- Postgres volume modes: fresh (per-task unique, concurrency-safe) or reuse (shared by dataset)

## Key techniques

### Sealed experiment identity
The `compute_sealed_hash()` function (seal.py:18-93) creates a deterministic hash over canonical JSON. This means two researchers running the same model, sampling, workflow, prompts, and kwargs will get IDENTICAL sealed hashes — and identical frozen specs will refuse to run if any input changes. The hash serves as both a reproducibility guarantee and a resume checkpoint key.

### Content-addressed freeze store with per-cell isolation
Each cell (task + attempt pair) gets its own git repo under `<freeze-root>/<sealed_hash>/<16-char-cell-token>/`. The cell token is SHA-256 of the Harbor trial name, so concurrent trials never share a git repo. This solves the problem that a sealed_hash-only CAS would create: two concurrent cells sharing one repo would contend for the same HEAD ref.

### Canonical JSON encoding for hashing
The seal function uses `json.dumps(payload, sort_keys=True, separators=(",",":"))` — sorted keys, minimal separators — before hashing. This ensures determinism across Python versions and JSON serialization libraries, since the canonical form strips whitespace and ordering variance.

### Frame-level boundary encoding for directory hashes
`resolve_solver_workflow_hash()` (resolvers.py:159-178) walks files in sorted POSIX-path order, frames each as `len(path):4 + path:utf-8 + len(content):8 + content`. The length prefix prevents boundary collisions — two files `a/bc` and `ab/c` would otherwise hash identically.

### Fail-closed agent kwargs
Both `build_inner_agent()` functions (claude.py:180-228, codex.py:164-188) check each incoming kwarg against a known-supported set and raise `SpacedockSolverAgentError` on unrecognized fields. This prevents silent dropping of experiment-critical settings — if a researcher specifies `max_turns: 500` for Codex (which only supports the default 200), the error message names the unsupported field and provides a hint.

### Three-layer leak protection
1. **Runtime blocking**: `RazorbackCodex.exec_as_agent()` intercepts commands before execution — blocks forbidden public lookup commands outright, wraps the outer `codex exec` with shell guards (custom PATH with wrappers for curl/wget), and installs PreToolUse hooks that gate every tool use.
2. **Post-hoc taint scanning**: `audit/taint.py` reimplements dataagentbench's scanner with razorback-specific divergence — only the four named canonical-data libraries are forbidden (not ALL pip installs), because generic compute libraries like rapidfuzz or duckdb are legitimate agent tools.
3. **Coverage verification**: dispatch manifests trace every subagent spawn. Missing manifests = coverage gaps. The audit flags these as `attempt_incomplete`.

### Running budget gate
Unlike most benchmark tools that track cost post-hoc, Razorback's budget gate (`budget.py`) uses `flock`-exclusive atomic writes (tempfile+rename+fsync) to manage a shared JSON ledger across concurrent invocations. The pre-launch check compares `current_total_usd + new_estimate > cap` and raises `BudgetExceededError` (exit code 22) BEFORE spending compute. This is critical for multi-invocation experiments where a researcher might launch 100 trials in parallel.

### Stratified pass@1 with honest CIs
The scoring system (verdict.py, aggregate.py) computes per-query Wilson CIs at the cell level (k correct out of n trials) but deliberately sets `wilson_ci: null` at the stratum level because the mean of per-query proportions is not a binomial proportion — it's a mean of binomials. This is mathematically honest where many benchmark tools would compute a misleading pooled CI.

### Plugin-driven benchmark generation
Benchmarks are not hardcoded. The `kind: harbor` + `plugin: dab` combination invokes `razorback-plugin-dab generate` as a subprocess, which emits harbor-shaped task directories. The plugin model means new benchmarks can be added without touching Razorback core — they just need to implement the task.toml contract and register an entry point.

### Harbor subprocess orchestration
`rk run` doesn't use Harbor's Python API directly for job execution; it serializes to `_job_config.yaml` and invokes `harbor run -c <yaml>` as a subprocess. This isolates Razorback from Harbor's process lifecycle and lets the aggregator work on filesystem state rather than in-memory objects. The HOME is staged under the runs-dir so Harbor's hardcoded `~/.cache/harbor` stays sandboxed.

## Design trade-offs

**Correctness over speed**: Every freeze pins every dynamic input. Every audit scan reads every trace line. This adds overhead (API calls for model resolution, docker inspect for digests, file reads for CLI hashing) but eliminates reproducibility failures that plague ad-hoc benchmark scripts.

**Strict schema over flexibility**: Every Pydantic model uses `extra="forbid"`. Every agent reveals unsupported kwargs with explicit error messages. Fail-closed means a researcher who upgrades razorback won't silently lose experiment controls — they'll get a clear error.

**Filesystem state over API objects**: The aggregator reads `result.json` and trial directory structures rather than using Harbor's in-memory `JobResult`. This works because Harbor runs as a subprocess and Razorback can't hold object references. The cost is that the aggregator must handle partial/malformed files gracefully and reconstruct stratum identity from multiple sources (view_manifest.json → config.json → trial dir name parsing).

**Harbor as infrastructure dependency**: Razorback builds on Harbor 0.6.6 (pinned) rather than implementing its own trial runner. This means it inherits Harbor's container lifecycle, trial management, and verifier contract. The subprocess invocation means Harbor version drift is detectable (rk run checks provenance.harbor_version) but not automatically resolved.

**Git-backed freeze trees**: The freeze tree uses git for checkpointing rather than a simpler file-copy model. This enables per-stage commits (setup/ready, run/before-agent, run/after-agent) and clean resume from any checkpoint, but adds git dependency and the complexity of concurrent-repo management.

**Single-agent Spacedock dispatch in benchmark context**: The current Codex first-officer path dispatches one worker per benchmark trial, not a full multi-stage workflow with gates and rejections. This is a deliberate simplification — the benchmark context doesn't need full Spacedock entity management — but it means the current path doesn't exercise the full Spacedock gate/rejection/reuse pipeline.

## Comparison to related projects

Unlike **MELT** (which evaluates agent memory lifecycle), Razorback evaluates agent TASK PERFORMANCE — pass@1 on benchmark datasets, with provenance that makes the run defensible. MELT asks "does the agent remember?"; Razorback asks "does the agent solve the task, and can we prove we didn't cheat?"

Unlike **FrontierCode** (which measures mergeability of PRs), Razorback is about reproducible measurement methodology. FrontierCode is a specific benchmark; Razorback is infrastructure for running ANY Harbor-compatible benchmark with cryptographic provenance.

Unlike **general agent harnesses** (Bram, Ruflo, Omnigent) which wrap agent execution, Razorback wraps BENCHMARK execution. It doesn't care what agent you use — it cares that the run is frozen, scored, audited, and comparable to other runs.

The sealed-hash approach is most similar to **content-addressed build systems** (Nix, Bazel) — inputs hash to a single identity, and identical inputs produce identical hashes. This is rare in ML benchmarking where ad-hoc scripts with undocumented environment dependencies are the norm.

Unlike **Akmon** (tamper-evident agent session records), Razorback seals the EXPERIMENT INPUTS rather than the session outputs. Akmon proves the agent session wasn't tampered with; Razorback proves the experiment can be recreated.

The author is CL Kao, who also created **Spacedock** (the first-officer/ensign multi-agent workflow system). Razorback is the bridge between Spacedock's orchestration model and Harbor's benchmark execution model — it's the tool that lets you run Spacedock workflow agents inside reproducible benchmarks. The `spacedock_solver` agent kind is the key integration point.

## Source analysis

Key files by line count and role:

| File | Lines | Role |
|------|-------|------|
| `translate.py` | 656 | Spec → Harbor JobConfig bridge |
| `spacedock_solver.py` | 609 | Outer orchestration agent |
| `run.py` (cli) | 423 | rk run command |
| `spec/schema.py` | 343 | Spec schema (Pydantic) |
| `runs/aggregate.py` | 770 | Post-run filesystem aggregator |
| `taint.py` | 588 | Post-hoc trace scanner |
| `diff/diff.py` | 176 | Paired run comparison |
| `codex.py` (_runtime) | 307 | Codex runtime adapter |
| `claude.py` (_runtime) | 237 | Claude runtime adapter |
| `budget.py` | 275 | Running-budget gate |
| `seal.py` | 111 | Sealed hash computation |
| `score/render.py` | 196 | Score output formatting |
| `score/verdict.py` | 103 | Against-constant verdicts |
| `provenance/resolvers.py` | 245 | Dynamic input resolvers |
| `provenance/freeze_cmd.py` | 183 | Freeze command |
| `packages/razorback-plugin-dab/` | ~600 | DAB plugin (12 datasets) |

Entry point: `rk = "razorback.cli:app"` — a Typer app with subcommands `run`, `freeze`, `score`, `audit`, `runs`, `constraints`, `baseline`, `registry`, `research`.
