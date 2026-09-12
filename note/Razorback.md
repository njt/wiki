# Razorback

Razorback is a Python CLI (`rk`) for running reproducible agentic benchmark experiments on top of Harbor. It treats benchmark runs as scientific experiments — every dynamic input is pinned to a cryptographic hash before execution, every trial trace is scanned for leakage after completion, and every score is computed with mathematically honest confidence intervals. The thesis is that agent benchmarking without provenance isn't science; you can't claim your model beats a baseline if you can't prove the run was frozen, the agent didn't cheat, and the budget wasn't exceeded. Built by CL Kao (the same mind behind Spacedock), it integrates Spacedock's first-officer/ensign multi-agent orchestration model into Harbor's containerized benchmark execution.

---

## Architecture

Razorback is a Python monorepo (~14,500 lines) layered on Harbor 0.6.6, Typer, and Pydantic. It operates as an orchestrator — it doesn't run benchmarks itself, it generates Harbor job configurations and invokes Harbor as a subprocess.

The pipeline flows through six stages:

```
spec.yaml → rk freeze → spec.frozen.yaml + provenance.yaml
spec.frozen.yaml → rk run → Harbor Job → run-dir/
run-dir/ → rk score → stratified pass@1 JSON
run-dir/ → rk audit → taint + coverage report
run-dir-A/ vs run-dir-B/ → rk runs diff → paired comparison
```

### Spec system (`src/razorback/spec/`)

The spec schema (`spec/schema.py`, 343 lines) uses discriminated unions for extensibility:

- **Agent kinds**: `nop` (no-op for testing), `claude-cli` (direct Claude wrapper), `codex` (direct Codex wrapper), `spacedock_solver` (Spacedock first-officer dispatch with sealed provenance and freeze/resume)
- **Benchmark kinds**: `local` (local task dirs), `harbor` (resolved from Harbor's package registry, with optional plugin), `harbor-local` (local Harbor-shaped dirs for dev iteration)

Every Pydantic model uses `extra="forbid"` — there is no silent field dropping.

### Agent system (three layers)

**Layer 1 — Runtime adapters** (`agents/_runtime/claude.py`, `agents/_runtime/codex.py`): Subclass Harbor's installed agents, adding Razorback-specific policy:
- Claude: tool allow/deny policy (shell-quoted comma-separated disallowed-tools list for safe flag rendering), plugin staging from host→container
- Codex: web search disabled, public-lookup shell guards (custom PATH with wrappers for curl/wget/npm, PreToolUse hooks that scan every command before execution), proxy env stripped during install, unsupported kwarg rejection

**Layer 2 — SpacedockSolverAgent** (`agents/spacedock_solver.py`, 609 lines): Outer wrapper providing:
- Sealed identity hash computation (model + sampling + workflow hash + prompt hashes + skill version + kwargs)
- Content-addressed freeze/resume store: each cell (task+attempt) gets its own git repo at `<freeze-root>/<sealed_hash>/<cell-token>/`
- Per-stage git commits (setup/ready, run/before-agent, run/after-agent) for clean resume
- Run instruction composition: first-officer role prompt + solver workflow README + task instruction
- Dispatch trace manifest emission (subagent dispatch count per trial)

**Layer 3 — Seal** (`agents/seal.py`): `compute_sealed_hash()` produces a deterministic 32-char hex hash from canonical JSON encoding (sorted keys, `(",",":")` separators, null values pinned not dropped). The hash serves as both reproducibility token and resume checkpoint key.

### Translate (`translate.py`, 656 lines)

The bridge from Razorback's spec to Harbor's `JobConfig`. Key paths:
- `spec_to_job_config()`: resolves agent config, benchmark tasks, and environment config
- `_build_agent_config()`: maps each agent kind to the correct import_path, resolves auth from `.env`, builds Harbor-compatible kwargs
- `_build_harbor()`: resolves Harbor datasets via `PackageDatasetClient`, invokes plugins via subprocess, handles benchmark-family-specific view materialization (spider2-dbt, swe-bench-pro)
- `_invoke_plugin_generate()`: runs `razorback-plugin-<name> generate` as a subprocess, collects emitted task dirs and trial-name maps

### Provenance & freeze (`provenance/`)

Resolves every dynamic input into a pinned value, writing `spec.frozen.yaml` and `provenance.yaml`:

- Model version → dated snapshot ID via Anthropic API (with retry for transient 502/503/504)
- Docker image digest via `docker image inspect`
- Agent CLI binary hash (SHA-256 of the binary on $PATH)
- Harbor version, git SHA, plugin entry-point inventory
- Solver workflow content hash (recursive dir hash with frame-level boundary encoding to prevent path collisions)

### Budget gate (`budget.py`)

A file-locked JSON ledger for multi-invocation experiments. Uses `flock`+`tempfile.rename`+`fsync` for atomic concurrent writes. Pre-launch gate: `current_total + new_estimate > cap → exit 22`. Handles subscription-auth null-cost by tracking `cost_known: false` and using pre-launch estimates as conservative proxy.

### Score (`score/`)

Per-query stratified pass@1. Wilson CIs at the cell level (k correct out of n trials IS a binomial proportion). Stratum CI is deliberately null because mean of per-query proportions is not binomial. Against-constant comparison yields matches/above/below point verdicts.

### Audit & taint (`audit/`)

Three scan categories, ported from dataagentbench with razorback-specific narrowing:
1. **forbidden_lookup**: curl/wget/npm/pip-install-of-canonical-data-libs/huggingface-cli/web_search — only the four named canonical-data libraries are forbidden, not all pip installs
2. **trace_coverage**: missing or partial subagent dispatch manifests, hook reconciliation failures
3. **attempt_incomplete**: timed-out workers, non-terminal frontmatter status

### Diff (`diff/`)

Paired comparison: per-query Wilson CIs per arm, exact McNemar p-values, paired bootstrap CI on stratified delta, power analysis (MDE at fixed N). Refuses cross-benchmark comparisons and mismatched seeds.

### DAB plugin (`packages/razorback-plugin-dab/`)

Reference plugin implementation for DataAgentBench. Generates Harbor task dirs with docker-compose sidecar databases. Supports 12 datasets, three workspace variants (direct-minimal, direct-structured, spacedock), and two query modes (batch, per-query). Discovered via `razorback.plugin_args` entry point.

## Key techniques

**Sealed experiment identity**: `compute_sealed_hash()` (seal.py) produces a deterministic hash from canonical JSON — model + sampling + workflow hash + prompt hashes + skill version + kwargs + task identity. Two researchers with identical inputs get identical hashes. The frozen spec refuses on hash mismatch.

**Per-cell freeze isolation**: Each concurrent trial gets its own git repo, keyed by `sealed_hash/cell_token` where the cell token is derived from the Harbor trial name. This prevents concurrent cells from contending for one shared `HEAD` ref — the earlier sealed_hash-only approach only worked with a single active writer.

**Frame-level content hashing**: Directory hashes encode `len(path):4 + path:utf-8 + len(content):8 + content` per file, preventing boundary collisions where files `a/bc` and `ab/c` would hash identically under naive concatenation.

**Fail-closed agent kwargs**: Both runtime adapters enumerate supported kwargs and raise on unrecognized fields. No silent dropping — if you set `max_turns: 500` on Codex (which only supports 200), you get an error with a hint about using timeout controls instead.

**Three-layer leak protection**: Runtime blocking (shell wrappers, PreToolUse hooks) → post-hoc scanning (forbidden command patterns in JSONL traces) → coverage verification (dispatch manifest completeness). Each layer catches what the previous can't — runtime can't catch what a worker types in a heredoc, but the taint scanner can.

**Running budget gate with atomic ledger**: Shared JSON file under exclusive lock (flock) + atomic write (tempfile+rename+fsync) — prevents corrupted state from concurrent writers and ensures the gate sees a consistent total before launch.

## Design decisions

**Correctness over speed**: Freeze pins every dynamic input (model version, image digest, binary hash, git SHA, plugin inventory). Audit scans every trace line. Overhead is real but eliminates whole classes of reproducibility failure.

**Strict schema over flexibility**: `extra="forbid"` everywhere. Explicit error messages for unsupported kwargs. The system refuses to run when inputs are ambiguous rather than silently choosing defaults.

**Filesystem over API**: The aggregator reads filesystem state rather than Harbor's in-memory objects because Harbor runs as a subprocess. This means the aggregator must handle partial/malformed files and reconstruct stratum identity from multiple sources.

**Harbor as infrastructure**: Builds on Harbor's container lifecycle and trial management rather than reimplementing. The subprocess boundary means Harbor version is tracked in provenance but can't be enforced beyond the drift check.

**Git-backed freeze trees**: Uses git for checkpoint commits (setup/ready, before-agent, after-agent) enabling clean resume. Adds git dependency but provides per-stage rollback that a file-copy model wouldn't.

## Comparison notes

Unlike [[MELT]] (which evaluates agent memory lifecycle), Razorback evaluates agent TASK PERFORMANCE with cryptographic provenance. MELT asks "does it remember?"; Razorback asks "did it solve it, and can we prove how we know?"

Unlike [[FrontierCode]] (a specific PR-mergeability benchmark), Razorback is benchmark INFRASTRUCTURE — it runs any Harbor-compatible benchmark with sealed provenance and audit.

Unlike agent harnesses ([[Bram]], [[Ruflo]], [[Omnigent|Introducing Omnigent]]) which wrap agent execution, Razorback wraps BENCHMARK execution. The agent is a configurable input, not the system being built.

The sealed-hash approach is closest to content-addressed build systems (Nix, Bazel) — an approach rarely applied to ML benchmarking, where ad-hoc scripts with undocumented environments are the norm.

The author, CL Kao, also created [[Manifest-Driven Development|Spacedock]]. Razorback is the bridge between Spacedock's first-officer/ensign orchestration model and Harbor's benchmark execution model. The `spacedock_solver` agent kind is where Spacedock's multi-agent dispatch runs inside a reproducible benchmark container.

---
*Sources: [[raw/razorback]]*
*Tags: #tool #benchmark #agents #cli #python*
*Last updated: 2026-07-18*
