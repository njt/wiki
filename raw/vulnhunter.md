---
url: https://github.com/capitalone/vulnhunter
title: VulnHunter
author: Capital One
date_fetched: 2026-07-18
date_published: 2025
---

# VulnHunter — Full Repo Analysis

Capital One's open-source agentic AI security tool for source code vulnerability hunting. Built as a suite of Claude Code skills (pure prompt engineering + Python helper packages) that form a closed-loop Hunt → Fix → Verify pipeline for security vulnerabilities.

## Repository Structure (3,800+ files, ~61K lines across Python tests)

Five self-contained subdirectories:

| Path | Lines | Language | Purpose |
|------|-------|----------|---------|
| `vulnhunt/` | ~2,500 (markdown prompts) | Prompt-only | Core scanner skill: SKILL.md + 11 phase files orchestrating a multi-agent hunt |
| `vulnhunter-fix/` | ~15,000+ (Python + prompts) | Python + Markdown | Fix skill: TDD-driven remediation, git worktree orchestration, PR delivery |
| `vulnhunt-fix-verify/` | ~1,500 (markdown) | Prompt-only | Read-only verification skill for fix correctness |
| `vulnhunter-agent/` | ~18,000 (Python, inc. tests) | Python | Headless runtime: Claude Agent SDK integration, GitHub automation, audit stream |
| `harness/` | ~3,000 (Python) | Python | Developer tooling: batch scanning + benchmarking against ground truth |

## Core Architecture

### The Skills Layer (Prompt Engineering)

The three Claude Code skills are prompt-only with no runtime code — pure instruction design:

**`/vulnhunt` (vulnhunt/SKILL.md + phases/)** — The orchestrator skill, 353 lines of orchestration logic + 10 phase files. Uses a strict orchestrator-worker pattern:
- Phase 1 (Recon): one subagent builds input inventory, subgraph partitions, and threat model
- Phase 2 (Hunt): parallel trace agents — 3 per partition (INJ/NAV/LOG class groups) + 1 sink-driven auditor. Minimum agent count = (3 × partition_count) + 1. Dispatched in waves of max 6 to prevent 429 API failures
- Phase 2b (Verify): adversarial falsification — a single subagent tries to disprove every candidate
- Phase 3a-c (Reproduce/Test/Fix): exploit PoCs + exploit tests built per confirmed finding
- Phase 3d (Sweep): variant discovery using "sweep patterns" to find sibling instances of confirmed vulns
- Phase 4 (Report): compiled from output files

Key design patterns:
- **Forward-trace paradigm**: Starts at attacker entry points and traces data forward to sinks, unlike traditional SAST which searches backward from sinks
- **Falsification engine (Phase 2b)**: "Historically, ~50% of candidate findings are false positives" — the verification phase is mandatory, adversarial, and explicitly tries to disprove each candidate
- **Subgraph partitioning**: Uses union-find on a two-level application call graph to partition code into non-overlapping trace scopes, preventing agent context contamination
- **Class-group specialization**: Three vulnerability class groups (Injection/INJ, Navigation-Auth/NAV, Logic-Crypto/LOG) with separate class-specific instruction files for each trace agent
- **HARD GATES**: Every candidate must pass 5 gates in order before being written up: Gate 0 (designed behavior?), Gate 1 (reachability), Gate 2a (attacker-controlled?), Gate 2b (effective sanitization?), Gate 3 (new capability gained?). Gate evaluation requires empirical source-code verification, not training-knowledge assumptions
- **The orchestrator stays lean**: Explicit instruction to not read result files into context — only verify file existence via Glob. Subagent return messages capped at ≤20 words

**`/vulnhunter-fix` (vulnhunter-fix/SKILL.md + references/)** — TDD-driven remediation with two modes (in-place and fork):
- Phase-boundary checkpoints: operator-gated approval between phases
- Git worktree isolation per finding cluster
- RED→GREEN evidence pipeline: exploit demo → failing test (with evidence persisted to disk as JSON) → fix → regression check
- Seven mechanical delivery gates before `gh pr create`: severity mask, body completeness, scope, idempotency, anti-merge math (group_cost ≤ 0.6 × split_cost), verification table, committed test naming
- Sweep algorithm: two-pass sibling defect detection (graph-anchored symbol pass + regex pattern pass)
- Collaboration loop for interactive in-place mode: pauses for developer input on blocking decisions
- 13 helper scripts (preflight, mode detection, worktree setup, parse, cluster scoring, PR body validation, test naming check, etc.)
- 12 reference files encoding patterns: CWE fix patterns, sweep patterns, repo-type adapters, approved crypto algorithms, remediation rigor rules, etc.

**`/vulnhunt-fix-verify` (vulnhunt-fix-verify/SKILL.md + phases/)** — Read-only independent verification:
- Strict tool envelope: Read/Write/Edit/Glob/Grep/Agent only — no Bash, no network
- Four-phase workflow: preflight → extract findings → per-VULN verification with 4 gates → emit verified JSON
- Schema-validated output against `verify_disposition.schema.json`
- "Fail-closed on schema drift" — code is the source of truth, not developer comments

### The Agent Runtime (Python)

`vulnhunter-agent/agent/` wraps the skills in a headless Python runtime (~3,400 lines of production code):

- **`__main__.py`** (1,320 lines): CLI with scan/verify modes, exhaustive flag validation, audit stream wiring
- **`runner.py`** (1,275 lines): Claude Agent SDK integration — builds prompt with pre-resolved metadata, manages SDK sessions with retry/continuation logic, handles cold-start 429 backoff (60s → 120s → 300s), detects stalled subagents (60 consecutive zero-progress continuations = abort)
- **`config.py`** (873 lines): TOML + env-var configuration with 14 frozen dataclass config sections (Anthropic, OAuth, TLS, Sandbox, Telemetry, Scan, GitHub, Publish, Issues, Verify, Logging, Audit, RepoProperties)
- **Auth**: `ApiKeyTokenManager` and `OAuthTokenManager` share a common `get_valid_token()` interface. Bedrock OAuth mode for enterprise proxy environments
- **Issues pipeline**: extract findings (via Haiku LLM call) → semantic dedup (via Sonnet) → render issue body → post to GitHub with label-based dedup pool
- **Audit stream**: JSONL emission (lifecycle events + per-finding observations) for downstream ingest
- **GitHub integration**: dual-token model (scan_token for clone+issues, reports_token for publish), broker mode for token-file consumption
- **Manifest**: `scan_manifest.json` validated against JSON schema, serves as stable integration contract for downstream automation

## Key Technical Insights

### 1. Prompt-as-Code with Deterministic Dispatch

The hunt skill treats prompts as executable code with precise dispatch rules. The orchestrator SKILL.md specifies exact agent counts (3 per partition + 1 sink-driven), wave dispatch of max 6 agents, and hard synchronisation barriers. This is not LLM autonomy — it's deterministic workflow expressed in natural language, closer to an Airflow DAG than a chatbot.

### 2. Falsification Over Confirmation

The verification phase (phase2b) is structurally adversarial: it tries to DISPROVE every candidate. This inverts the typical security scanner approach. The explicit claim that ~50% of candidates are false positives sets a realistic baseline. The consensus-skip shortcut (if all agents for a partition cite the same elimination evidence at the same file:line, skip re-verification) is a pragmatic efficiency hack.

### 3. Subgraph Partitioning via Union-Find

Phase 1 builds a two-level call graph per entry point, catalogs shared infrastructure (modules imported by >50% of entry points), then uses union-find to compute subgraph partitions. Two entry points are in the same partition if they share any non-infrastructure function. This is a genuinely novel approach to context scoping for parallel LLM agents — each trace agent gets a self-contained slice with no cross-contamination.

### 4. Cold-Start Rate Limit Detection

The runner distinguishes between cold-start and mid-stream transient errors. Consecutive 429/5xx SystemMessages BEFORE the first AssistantMessage trigger session restart with backoff. After the first AssistantMessage arrives, transients are logged but don't restart (the harness's subprocess-level retry handles those). The stall detector (60 consecutive continuations with zero task lifecycle events) is an abort mechanism, not a retry — it catches genuinely hung agents.

### 5. The Anti-Merge Math

When grouping multiple vulnerability fixes into one PR, the fixer applies a mechanical check: group_cost ≤ 0.6 × split_cost (measured in distinct files touched). This prevents over-aggressive grouping that would make PRs unreviewable. The either/or on source vs test files handles the common "tests mirror source tree" convention cleanly.

### 6. Dual Auth for GitHub

The agent uses two distinct GitHub tokens (scan_token for clone+issues, reports_token for publish) rather than one. This is deliberate defense-in-depth: a compromised scan token (which touches arbitrary target repos) can't also push to the results repository. The broker mode allows an external parent process to rotate tokens onto disk, keeping the agent itself stateless.

## Design Trade-offs

- **Opus-only**: The skills hard-gate on Opus 4.7+. Sonnet/Haiku produce "noticeably worse results: weaker clustering, hand-wavier fix proposals, more dead-end iterations." This trades broad model compatibility for output quality.
- **Prompt-only skills vs. code**: The hunt and verify skills are pure markdown with zero runtime code. This makes them inspectable and auditable, but means version control is the only change management mechanism — no type checking, no unit tests for the instructions themselves.
- **Synchronous pipeline with human gates**: The fixer runs unattended within a phase but pauses at every phase boundary for operator approval. This deliberately prevents fully-automated fix application — a safety choice that trades speed for oversight.
- **Read-only as default**: By default, scans run without Bash — exploit tests are written but not executed. Code execution requires explicit --enable-bash + --no-read-only flags, with Bash intentionally absent from the config-file allow-list so "a stray TOML can't silently re-enable arbitrary code execution."
- **Headless via Claude Agent SDK**: The runtime uses the SDK (not CLI subprocess), giving direct access to streaming events for monitoring, retry logic, and cost tracking. This trades the simplicity of shelling out to `claude` for fine-grained observability.

## Comparison to Related Projects

Unlike traditional SAST tools (Semgrep, CodeQL) which pattern-match on syntax, VulnHunter reasons about data flow provenance — whether attacker-controlled data actually reaches a sink. Unlike LLM-based code reviewers ([[OpenCodeReview]], [[Metis — ARM AI Security Code Review]]) which review surface-level changes, VulnHunter performs deep forward-trace analysis from every entry point.

The multi-agent orchestration is philosophically similar to Cloudflare's code review system ([[Orchestrating AI Code Review at Scale]]) but the coordination pattern is inverted: Cloudflare runs specialized agents in parallel then merges via a judge; VulnHunter partitions the codebase first, then dispatches each partition to parallel class-group agents, with a separate adversarial verification pass.

The TDD-driven fix pipeline (exploit → failing test → fix → verify) shares DNA with [[no-mistakes]]'s validation pipeline but is security-specific rather than general-purpose.

The fix verifier's read-only "code is the source of truth" discipline parallels [[Interdict]]'s approach of validating agent actions against real state, though applied to vulnerability fixes rather than database queries.
