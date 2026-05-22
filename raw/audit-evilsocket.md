---
url: https://github.com/evilsocket/audit
title: "audit: Cloudflare-style 8-stage vulnerability discovery agent"
author: evilsocket (Simone Margaritelli)
date_fetched: 2026-05-22
date_published: 2026-05
---

# audit — Full Repo Analysis

## Overview

`audit` is a Python agent harness that implements Cloudflare's Project Glasswing vulnerability discovery pipeline using the Claude Code Agent SDK. It uses Claude Pro/Max subscription billing (not the metered API), runs 8 sequential stages through SQLite-backed state management, and produces schema-validated JSON outputs at every step. MIT-licensed, ~1K lines of Python.

## Architecture

### Pipeline topology

The orchestrator (`audit/orchestrator.py`, 145 lines) drives a fixed sequence with two bounded loops:

```
Recon → (Hunt → Validate → Gapfill)* → Dedupe → Trace → Feedback → (Hunt → Validate → Dedupe → Trace)* → Report
```

The outer loop (Gapfill) runs `gapfill_iterations + 1` times (default 2), re-queueing under-covered areas. The inner loop (Feedback) runs once after Trace, converting reachable findings into new Hunt tasks for pattern propagation.

### Core abstractions

1. **StageContext** (`audit/stages/_common.py`): A dataclass holding run_id, repo_path, config, optional live_target, and scope_notes. Provides path resolution for prompts, schemas, results, and work directories.

2. **StateDB** (`audit/state.py`, 472 lines): SQLite DAO with tables for runs, tasks, findings, traces, dedupe_groups, costs, and artifacts. Uses `row_factory = sqlite3.Row` for dict-like access. All writes go through explicit commit().

3. **AgentResult** (`audit/runner.py`): A dataclass capturing payload, cost, token usage, session_id, and artifact_path from each agent invocation.

4. **StageConfig** (`audit/config.py`): Per-stage model, concurrency, tools, max_turns, permission_mode, and repair_attempts loaded from `config/stages.yaml`.

### Stage implementations

Each stage module in `audit/stages/` exports a single async function. The pattern is:
- Query StateDB for pending work
- Dispatch concurrent agents via `asyncio.Semaphore` (concurrency varies by stage)
- Each agent call goes through `run_agent()` in `runner.py`
- Results go back into StateDB as tasks, findings, traces, costs, and artifacts

**Stage 1 - Recon** (`recon.py`, 58 lines): Single agent (Opus 4.7, 60 max turns). Maps the repo into subsystems, entry points, trust boundaries, and external inputs. Mines git history for past security patches as sibling-bug indicators. Emits 30-80 narrowly-scoped hunt tasks.

**Stage 2 - Hunt** (`hunt.py`, 113 lines): Concurrent agents (Sonnet 4.6, default concurrency 50). Each agent gets ONE attack class and ONE scope. Runs in per-task scratch directories. Can compile/run PoCs. Emits findings with line-level evidence and optional PoC code.

**Stage 3 - Validate** (`validate.py`, 93 lines): Adversarial review (Opus 4.7 — deliberately different model). Agents can only disprove, not generate new findings. Each emits confirmed/rejected/needs_more_info with mandatory alternative explanation.

**Stage 4 - Gapfill** (`gapfill.py`, 114 lines): Coverage analysis (Sonnet 4.6). Builds subsystem x attack_class matrix from completed tasks, identifies under-covered cells, emits new Hunt tasks.

**Stage 5 - Dedupe** (`dedupe.py`, 70 lines): Clustering (Sonnet 4.6). Groups confirmed findings by root cause — "same patch would fix both." Picks canonical member per group by PoC success > severity > confidence.

**Stage 6 - Trace** (`trace.py`, 85 lines): Reachability analysis (Opus 4.7). Backward traces from sink to external entry point. Records call chains, blockers (sanitizers, auth gates, dead code), and external inputs. The gating stage — only reachable findings ship in the report.

**Stage 7 - Feedback** (`feedback.py`, 65 lines): Pattern propagation (Sonnet 4.6). Extracts transferable patterns from reachable traces, greps the codebase for structurally similar call sites, emits new Hunt tasks targeting siblings.

**Stage 8 - Report** (`report.py`, 116 lines): Final document (Sonnet 4.6). Takes confirmed + reachable + canonical findings, emits schema-validated JSON with title, severity, CWE, evidence, trace, and concrete recommendation.

### Runner infrastructure

`audit/runner.py` (386 lines) wraps `claude-agent-sdk`'s `ClaudeSDKClient`:

- **Schema injection**: Appends the full JSON schema to the system prompt so the model never guesses field names. This drastically reduces first-attempt validation failures.
- **Repair turn**: On schema validation failure, sends the model back the errors and asks for a fix. One repair attempt per stage by default, two for Recon (critical one-shot) and Report (MUST validate).
- **API error classification** (`_classify_api_error`): String-matches error text to distinguish quota_exhausted (QuotaExhaustedError, terminal) from transient (TransientAgentError, retry with exponential backoff: 30s base, 240s cap, 3 retries).
- **JSONL artifacts**: Every message exchanged (user, assistant, tool use, tool result, repair, final payload) is written to a JSONL file for debugging and audit trails.
- **JSON extraction** (`audit/json_utils.py`, 113 lines): Three-pass strategy — try full text as JSON, try ```json fenced block, try largest balanced {…} or […] substring.

### Auth module

`audit/auth.py` (149 lines) implements Claude Code's auth precedence list:
1. Gateway mode (ANTHROPIC_BASE_URL pointing away from anthropic.com + ANTHROPIC_AUTH_TOKEN)
2. OAuth token (CLAUDE_CODE_OAUTH_TOKEN from `claude setup-token`)
3. Keychain login (~/.claude/.credentials.json from `claude login`)

Critically, it scrubs ANTHROPIC_API_KEY from the environment in subscription modes to prevent silent routing around OAuth.

### CLI

`audit/cli.py` (287 lines): Click-based CLI with commands: auth-check, run, status, report. The `run` command assembles config, auth, and live-target params, then calls `asyncio.run(run_pipeline(...))`.

## Prompts

Each of the 8 prompts in `prompts/` is a markdown file structured identically: Role, Objective, Inputs (with JSON schema example), Tools available, Output (pointing to schema), Method (numbered steps), Constraints. The prompts are loaded verbatim as system prompts — the schema is appended programmatically by the runner.

Key prompt engineering patterns:
- **Adversarial framing**: Validate prompt says "You are paid in rejected findings, not confirmed ones." This is the "deliberate disagreement" mechanism.
- **Conservatism**: Hunt prompt says "Be conservative with severity — never invent a 'high' to make the queue feel productive."
- **Hedged language detection**: The finding schema has a `hedged_language` boolean field — prompts instruct hunters to set it true if they use "might"/"could"/"possibly."
- **Narrow scope enforcement**: Recon prompt prohibits generic attack classes, requires concrete `scope_hint` naming the trust boundary above the sink.
- **Git history mining**: Recon prompt gives specific git commands for finding past security patches and instructs to grep for the same idiom in sibling files.

## Schemas

9 JSON Schema files in `schemas/` using Draft 7 with `additionalProperties: false` on every object. Key design choices:

- **$ref for shared types**: `recon_output.schema.json` references `hunt_task.schema.json` via `$ref`. The `json_utils.py` validator builds a referencing Registry from all sibling `.schema.json` files.
- **Strict ID patterns**: `finding_id` enforces `/^f_[a-z0-9_-]{1,64}$/`, `task_id` enforces `/^[a-z0-9_-]{1,64}$/`.
- **Enum-constrained fields**: `severity` is `["critical","high","medium","low","informational"]`, `verdict` is `["confirmed","rejected","needs_more_info"]`.
- **Semantic fields**: `hedged_language` flag on findings, `gaps_observed` array for coverage feedback, `blockers` on traces for documenting why a path is infeasible.

## Config

`config/stages.yaml` (50 lines) defines defaults (max_turns: 25, permission_mode: acceptEdits, repair_attempts: 1) and per-stage overrides. Key settings:

| Stage | Model | Concurrency | Tools | Max Turns | Repair Attempts |
|-------|-------|-------------|-------|-----------|-----------------|
| recon | claude-opus-4-7 | 1 | Read,Grep,Glob,Bash | 60 | 2 |
| hunt | claude-sonnet-4-6 | 50 | Read,Grep,Glob,Bash | 25 | 1 |
| validate | claude-opus-4-7 | 10 | Read,Grep,Glob | 25 | 1 |
| gapfill | claude-sonnet-4-6 | 1 | Read,Grep,Glob | 25 | 1 |
| dedupe | claude-sonnet-4-6 | 1 | Read | 25 | 1 |
| trace | claude-opus-4-7 | 10 | Read,Grep,Glob,Bash | 25 | 1 |
| feedback | claude-sonnet-4-6 | 1 | Read,Grep,Glob | 25 | 1 |
| report | claude-sonnet-4-6 | 1 | Read | 25 | 1 |

Bounded loops: gapfill_iterations=2, feedback_iterations=1.

Note: Validate uses a different model from Hunt (Opus vs Sonnet) — this is load-bearing for the "deliberate disagreement" pattern. Trace also uses Opus — the blog calls it "the stage that matters most."

## Key techniques

### Schema-as-system-prompt

The runner appends the literal JSON Schema to every agent's system prompt (`runner.py:191-198`). This means the model sees required fields, enum values, and `additionalProperties: false` before generating output. Combined with the repair turn, this achieves near-100% schema compliance on the first attempt.

### Three-pass JSON extraction

`json_utils.py`'s `extract_json()` tries: (1) parse full text as JSON, (2) find ```json fence, (3) find largest balanced brace/bracket substring with string-aware depth tracking. The third pass handles models that interleave JSON with prose despite instructions not to.

### Transient error classification with string markers

Rather than parsing structured error codes, `_classify_api_error()` matches substrings against known quota and transient markers. Unknown errors default to transient (retryable) — better to retry once than abort on a classification miss.

### Budget guard cooperative abort

The budget check fires between AND within stages. Hunt tasks check budget before starting; if exceeded, they set an `asyncio.Event` that causes all remaining tasks to skip. This prevents running 30 more tasks past the cap.

### Git history mining for sibling bugs

Recon's prompt instructs the agent to grep git history for security patches, then grep the rest of the codebase for the same vulnerable idiom in sibling files. This is a zero-cost heuristic on repos without that pattern, but catches real cross-component bugs on repos that have it.

### Live-target reproduction

When `--target-url` is set, Hunt reproduces findings via actual HTTP requests, Validate rejects non-reproducing findings, and Trace confirms reachability with real round-trips. Egress is restricted to the target host + 127.0.0.1.

### Coverage matrix (Gapfill)

Gapfill builds a `subsystem x attack_class` matrix from completed tasks, then emits new tasks for unexamined cells. This counters the "once SQL injection lands, the next twenty hunts all look like SQL injection" drift.

## Design trade-offs

**Optimized for: finding real bugs, not covering code.** The pipeline gates on reachability — most "is this code buggy?" findings are noise unless an attacker can actually reach the sink. This means false negatives (missing real-but-unreachable-from-outside bugs) are accepted to eliminate false positives.

**Sacrificed: cost predictability.** At 15-50 Hunt tasks and 25+ findings to validate at default concurrency (50 for Hunt), cost can spike. The `--max-cost-usd` flag and `--max-recon-tasks` cap are mitigation, not solution.

**Sacrificed: execution time.** The pipeline is sequential by design — Recon must finish before Hunt starts, etc. Parallelism exists within stages (concurrent Hunt tasks, concurrent Validations) but not between stages.

**Sacrificed: auto-remediation.** Unlike some security agents that attempt to generate patches, audit stops at reporting. The Cloudflare blog notes that letting the model write patches produced fixes that "quietly broke something else." The gap between "here's the exploit" and "here's the safe fix" remains human-mediated.

**Unusual choice: no sandbox at the harness level.** The README explicitly warns that Hunt agents are not OS-sandboxed and recommends running inside a disposable VM or container. This is a deliberate simplicity trade-off — the harness trusts the operator to provide isolation.

## Comparison to Cloudflare's original

Cloudflare's Project Glasswing post describes the architecture; audit ships the runnable code. Key differences:
- Cloudflare used Mythos Preview (restricted model); audit uses Claude Code Agent SDK with subscription billing
- Cloudflare had 9 stages; audit has 8 (combining or omitting one)
- Cloudflare operated at Cloudflare scale (50+ repos); audit is a single-repo tool
- Cloudflare's harness was internal; audit is MIT-licensed and publicly runnable

## Comparison to AISLE's approach

[[AI Cybersecurity After Mythos — The Jagged Frontier]] argues that cheap open-weights models can detect the same bugs as Mythos and that "the moat is the scaffold." audit embodies this thesis — it's a scaffold that happens to use Claude models, but the architecture is model-agnostic (supports OpenRouter, custom gateways, any model speaking the Anthropic Messages API).

## File listing

```
audit/__init__.py          (3 lines)   - Package init
audit/__main__.py          (4 lines)   - Entry point
audit/auth.py              (149 lines) - OAuth + API key scrubbing
audit/cli.py               (287 lines) - Click CLI
audit/config.py            (70 lines)  - YAML config loader
audit/json_utils.py        (113 lines) - JSON extraction + schema validation
audit/orchestrator.py      (145 lines) - Pipeline driver
audit/runner.py            (386 lines) - ClaudeSDKClient wrapper
audit/state.py             (472 lines) - SQLite DAO
audit/stages/__init__.py   (22 lines)  - Stage exports
audit/stages/_common.py    (75 lines)  - StageContext + helpers
audit/stages/recon.py      (58 lines)  - Stage 1
audit/stages/hunt.py       (113 lines) - Stage 2
audit/stages/validate.py   (93 lines)  - Stage 3
audit/stages/gapfill.py    (114 lines) - Stage 4
audit/stages/dedupe.py     (70 lines)  - Stage 5
audit/stages/trace.py      (85 lines)  - Stage 6
audit/stages/feedback.py   (65 lines)  - Stage 7
audit/stages/report.py     (116 lines) - Stage 8
config/stages.yaml         (50 lines)  - Per-stage config
prompts/01-recon.md                    - Recon system prompt
prompts/02-hunt.md                     - Hunt system prompt
prompts/03-validate.md                 - Validate system prompt
prompts/04-gapfill.md                  - Gapfill system prompt
prompts/05-dedupe.md                   - Dedupe system prompt
prompts/06-trace.md                    - Trace system prompt
prompts/07-feedback.md                 - Feedback system prompt
prompts/08-report.md                   - Report system prompt
schemas/recon_output.schema.json       - Recon output schema
schemas/hunt_task.schema.json          - Hunt task schema
schemas/finding.schema.json            - Finding schema
schemas/validation.schema.json         - Validation verdict schema
schemas/gapfill_output.schema.json     - Gapfill output schema
schemas/dedupe_output.schema.json      - Dedupe output schema
schemas/trace.schema.json              - Trace schema
schemas/feedback_output.schema.json    - Feedback output schema
schemas/report.schema.json             - Final report schema
tests/test_auth.py                     - Auth tests
tests/test_config.py                   - Config tests
tests/test_json_utils.py               - JSON extraction tests
tests/test_runner_errors.py            - API error classification tests
tests/test_schemas.py                  - Schema validation tests
tests/test_stage_context.py            - StageContext tests
tests/test_state.py                    - StateDB roundtrip tests
tests/fixtures/vulnerable_app/         - Test fixture (Flask app with known vulns)
```
