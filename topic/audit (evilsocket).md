# audit (evilsocket)

A runnable, MIT-licensed implementation of Cloudflare's Project Glasswing vulnerability discovery pipeline, driven by Claude Code Agent SDK with subscription billing. Eight sequential stages — Recon, Hunt, Validate, Gapfill, Dedupe, Trace, Feedback, Report — each a single markdown prompt + JSON schema, orchestrated through SQLite state management and concurrent agent dispatch. The key insight: many narrow agents in deliberate disagreement find more real bugs than one big model asked to "find everything."

---

## Architecture

The pipeline is driven by `audit/orchestrator.py:23` — a single async function `run_pipeline()` that sequences 8 stages with two bounded loops:

```
Recon → (Hunt → Validate → Gapfill)* → Dedupe → Trace → Feedback → (Hunt → Validate → Dedupe → Trace)* → Report
```

The outer Gapfill loop re-queues under-covered areas (default 2 iterations). The inner Feedback loop propagates reachable bug patterns to sibling code (default 1 iteration). Every stage writes to `StateDB` (`audit/state.py:155`), a SQLite DAO with tables for runs, tasks, findings, traces, dedupe groups, costs, and artifacts.

Each stage module in `audit/stages/` is ~50-115 lines. The pattern: query StateDB for pending work → dispatch concurrent agents via `asyncio.Semaphore` → each agent call goes through `run_agent()` in `audit/runner.py:103` → results flow back into StateDB.

**Stage 1 - Recon** (`recon.py`): Single Opus 4.7 agent (60 max turns). Maps repo into subsystems, entry points, trust boundaries. Mines git history for past security patches to seed sibling-bug tasks. Emits 30-80 narrowly-scoped hunt tasks.

**Stage 2 - Hunt** (`hunt.py`): Concurrent Sonnet 4.6 agents (default 50 concurrent). Each gets ONE attack class and ONE scope. Can compile/run PoCs in per-task scratch dirs. Emits findings with line-level evidence.

**Stage 3 - Validate** (`validate.py`): Adversarial review using Opus 4.7 — deliberately different from Hunt's Sonnet. Agents can only disprove, not generate new findings. Mandatory alternative explanation on every verdict.

**Stage 4 - Gapfill** (`gapfill.py`): Builds a subsystem x attack_class coverage matrix, identifies under-examined cells, emits new Hunt tasks. Counters the drift where "once SQL injection lands, the next twenty hunts all look like SQL injection."

**Stage 5 - Dedupe** (`dedupe.py`): Clusters confirmed findings by root cause — "same patch would fix both." Groups by shared helper function or identical defect, not similar symptoms.

**Stage 6 - Trace** (`trace.py`): Opus 4.7 backward traces from sink to external entry point. The gating stage — only reachable findings ship in the report. Records call chains frame-by-frame with real file/function/line verification.

**Stage 7 - Feedback** (`feedback.py`): Extracts transferable patterns from reachable traces, greps the codebase for structurally similar call sites, emits new Hunt tasks targeting siblings.

**Stage 8 - Report** (`report.py`): Schema-validated JSON with title, severity, CWE, evidence, trace, and concrete remediation per finding. Falls back to a minimal report if the agent fails.

The runner (`audit/runner.py`) wraps `ClaudeSDKClient` with three infrastructure pieces: schema injection into system prompts (eliminates field-name guessing), a repair turn on validation failure, and error classification that distinguishes quota-exhausted (terminal) from transient (retry with exponential backoff: 30s base, 240s cap, 3 retries).

## Key Techniques

**Schema-as-system-prompt**: `runner.py:191-198` appends the literal JSON Schema to every agent's system prompt. The model sees required fields, enum values, and `additionalProperties: false` before generating output. Combined with the repair turn, this achieves near-100% schema compliance on first attempt — critical when a schema failure on Recon means re-running the most expensive stage.

**Three-pass JSON extraction**: `json_utils.py:18` tries full-text parse, then ```json fence extraction, then string-aware balanced bracket scanning. The third pass handles models that interleave JSON with prose despite instructions — a common failure mode the repair turn can't fix (it only fires after parseable-but-invalid JSON).

**Adversarial framing in prompts**: Validate's prompt says "You are paid in rejected findings, not confirmed ones" and requires a mandatory `alternative_explanation` even when confirming. This is the prompt-engineering equivalent of Cloudflare's "deliberate disagreement" — structural skepticism beats "be more careful."

**Hedged language detection**: The finding schema (`schemas/finding.schema.json`) includes a `hedged_language` boolean. Hunt's prompt instructs agents to self-report when they use "might"/"could"/"possibly." This is a lightweight triage signal — hedged findings get lower effective confidence without needing a separate classifier.

**Git history mining**: Recon's prompt (`prompts/01-recon.md:77-82`) gives exact git commands for finding past security patches, then instructs grepping for the same vulnerable idiom in sibling files. Zero cost on repos without that pattern; catches real cross-component bugs on repos that have it.

**Live-target reproduction**: When `--target-url` is set, Hunt reproduces findings via HTTP, Validate rejects non-reproducing findings, and Trace confirms reachability with real round-trips. Egress is restricted to the target host + localhost only.

**Budget guard cooperative abort**: `orchestrator.py:60-68` checks cumulative cost between AND within stages. Hunt tasks check budget before starting; if exceeded, an `asyncio.Event` causes all remaining tasks to skip — preventing 30 more tasks from running past the cap.

## Design Decisions

**Reachability as the gate, not bug existence.** Most vulnerability scanners find sinks. audit only ships findings where an attacker-controlled input can actually reach the sink from outside the system. This accepts false negatives (real bugs unreachable from outside) to eliminate false positives. The Trace stage uses Opus 4.7 — the most expensive model — because the blog calls it "the stage that matters most."

**Model diversity is load-bearing.** Hunt uses Sonnet 4.6; Validate uses Opus 4.7. This isn't cost optimization — it's the "deliberate disagreement" rule from the Cloudflare paper. A model reviewing its own output has blind spots; a different model reviewing different model output catches more noise. Trace also uses Opus because reachability analysis requires the strongest reasoning.

**No auto-remediation.** Unlike some security agents that attempt to generate patches, audit stops at reporting. The Cloudflare blog documents why: letting the model write patches produced fixes that "quietly broke something else." The pipeline finds and confirms; the human writes the fix.

**No OS-level sandboxing.** The README explicitly warns that Hunt agents have Bash and are not sandboxed. The harness trusts the operator to provide isolation (VM, container). This is a deliberate simplicity trade-off — the harness is 1K lines of Python, not a sandboxing platform.

**Prompts are the source code.** The 8 markdown files in `prompts/` are the real intellectual property. The Python harness is thin orchestration — loading prompts, dispatching agents, validating schemas, storing state. Changing the pipeline's behavior means editing prompts, not code.

**SQLite as the backbone.** Every piece of state — runs, tasks, findings, traces, dedupe groups, costs, artifacts — lives in a single SQLite file (`state.db`). This gives resume support, cost tracking, and queryability without external infrastructure. JSONL artifacts in `results/` are the source of truth for raw agent output; the DB is the queryable index.

## Comparison Notes

**vs. Cloudflare's original Glasswing**: Cloudflare described the architecture using Mythos Preview (restricted model) against 50+ internal repos. audit packages the same pipeline as a runnable, model-agnostic tool using Claude Code subscription billing. The prompts, schemas, and orchestrator are open-source; the model access is the user's responsibility.

**vs. AISLE's approach** ([[AI Cybersecurity After Mythos — The Jagged Frontier]]): Fort argues "the moat is the scaffold, not the model." audit is that scaffold — it happens to use Claude models by default, but the architecture supports any model speaking the Anthropic Messages API (OpenRouter, custom gateways, cloud providers). The harness design doesn't assume Mythos-level capability.

**vs. Trail of Bits' Trailmark** ([[Trailmark]]): Trailmark builds queryable code graphs for security analysis. audit uses agents that directly read and grep the repo — no intermediate graph representation. The trade-off is that agents have richer context (they can read arbitrary files) but less systematic coverage.

**vs. general-purpose coding agents**: audit is narrow by design. Each Hunt agent sees one attack class and one scope. This is the opposite of "point a smart model at a repo and ask what's wrong." The harness, not the model, provides coverage through parallelism and gap analysis.

---

*Sources: [[summary/audit-evilsocket]]*
*Source URL: https://github.com/evilsocket/audit*
*Author: evilsocket (Simone Margaritelli)*
*Last updated: 2026-05-22*

#tool #security #agent-pipeline #vulnerability-discovery #multi-agent
