---
url: https://github.com/Moai-Team-LLC/agentic-product-standard
title: The Agentic Product Standard v2.0
author: Moai Team LLC (Alex Duch)
date_fetched: 2026-06-09
date_published: 2026-06-06
topics:
  - guardrails-and-feedback-loops
  - specifications-as-the-product
---

# The Agentic Product Standard v2.0 — Full Analysis

## What it is

A canonical standard for building production-grade agentic products, distilled from the production practices of Anthropic, OpenAI, Cognition, Sierra, and LangChain (2024–2026). It is NOT code — it is a specification repo with two tracks: (1) prose standards (STANDARD.md, AGENT_STANDARD.md) for humans to read, and (2) Claude Code skills (skills/) that operationalize the standard as in-editor behavior. The skills use a master-router + sub-skills architecture for progressive disclosure.

## Project structure

```
agentic-product-standard/
├── STANDARD.md                          ← canonical standard (product level), 451 lines
├── AGENT_STANDARD.md                    ← single-agent operational standard, 1597 lines
├── SCORECARD.md                         ← M0–M3 self-assessment mapped to Autonomy Ladder
├── CONTEXT.md                           ← shared vocabulary every skill speaks, 72 lines
├── CHANGELOG.md                         ← v1.0 → v2.0.0 (June 2026)
├── GOVERNANCE.md                        ← maintainer model, ADR-locked canons
├── ROADMAP.md                           ← Now / Next / Later
├── setup.sh                             ← quick setup: skills + optional AgenticMind
├── templates/security/                  ← red-team kit (lethal-trifecta, injection, MCP pin)
├── templates/ci/eval-gate.yml           ← CI workflow blocking merges on eval regression
├── examples/agenticmind-case-study.md   ← reference implementation compliance audit
├── docs/adr/                            ← architecture decision records (2 ADRs)
└── skills/                              ← Claude Code skill set
    ├── agent-builder/                   ← single-agent track (bundles AGENT_STANDARD.md + templates)
    └── agentic-product-architect/       ← multi-agent track: master router + 11 sub-skills
```

## Architecture pattern: Master-skill routing with progressive disclosure

The skill architecture (ADR-0001) uses a **master router skill** (`agentic-product-architect`) that classifies user intent and dispatches to exactly the relevant sub-skill(s). This is progressive disclosure — only the needed depth enters context. The master stays thin; sub-skills are self-contained, reference CONTEXT.md for shared terms, and evolve independently. Routing quality is an explicit maintenance obligation.

ADR-0002 establishes the two-representation design: STANDARD.md is the canonical prose (human-first, citable), while skills/ are the operators (agent-first, behavioral). CONTEXT.md bridges them with a shared vocabulary.

## The six principles (as of v2.0)

1. **Determinism by default, agency by necessity** — autonomy earned on evals, not granted upfront
2. **Architecture beats framework** — patterns outlive libraries
3. **Harness > model** — 98% of reliability lives in code around the LLM
4. **Context engineering is the core discipline** — what enters the context window determines everything
5. **Eval-driven development is non-negotiable** — no measurement, no improvement
6. **Security is a structural property, not a guardrail** — added in v2.0

## The canonical models

### Autonomy Ladder (L0–L4)
- L0: Single LLM call (classify/extract/summarize)
- L1: Augmented LLM (+ retrieval, + tools, + memory)
- L2: Workflow (deterministic code orchestrates LLM steps)
- L3: Orchestrator-Worker (LLM dynamically decomposes within bounded graph)
- L4: Autonomous Agent Loop (LLM chooses next step until termination)

**Escalation rule**: do not climb to L+1 until L delivers ≥90% pass rate on curated evals.

### Five Composition Patterns
1. Prompt Chaining (sequential decomposition)
2. Routing (classifier → specialist dispatcher)
3. Parallelization (fan-out + aggregation)
4. Orchestrator-Workers (central planner + dynamic workers)
5. Evaluator-Optimizer (generator + critic loop until acceptance)

**Meta-principle**: compose these in deterministic code first. Full agent loop is the last resort.

### 8-Layer Harness (was 7 in v1.x)
1. Agent Loop (gather → act → verify)
2. Context & Memory Management
3. Durable Execution (Workflow + Activity)
4. Guardrails (input/output validation, defense in depth)
5. Human-in-the-Loop (notify/ask/review)
6. Evaluation Layer (CI gates)
7. Observability & Tracing
8. Security & Identity (CROSS-CUTTING, added v2.0)

Layer 8 is cross-cutting — identity, least privilege, and isolation constrain every layer beneath them. Plus a Layer 9 (Cost & FinOps) added in v2.0.

### Cycle of Trust
gather context → propose action → check permissions → verify preconditions → execute → verify outcome → log trace → update memory

Permissions enforced in **code, never by prompt**. The Replit 2025 incident (agent wiped 1,200+ companies' DBs despite prompt "code freeze") is the canonical proof.

## Key techniques and innovation points

### 1. The 40% rule as harness doctrine

Keep context-window utilization below ~40% of the model's limit. Degradation past that threshold is non-linear — this is backed by Chroma's "context rot" research and Databricks' retrieval studies. Bigger windows don't repeal the rule. The frontier technique is **just-in-time retrieval** (Claude Code's glob+grep+read pattern) over precomputed vector RAG.

### 2. Bitter-pilled maintenance

From Daniel Miessler's PAI (MIT-licensed): tag every rule as **anti-fragile** (keep: verification harnesses, eval sets, data pipelines, tool contracts) or **fragile** (cut or re-test: chain-of-thought orchestrators, output-format parsers, retry cascades). Test: "Would a smarter model make this rule unnecessary?" If yes, it's scaffolding — remove it.

### 3. Closed enumerations over open vocabularies

For any rule the model has shown willingness to satisfy cosmetically (selecting from a category, naming a capability), inline the **complete allowed set** in the context read at runtime. A pointer to another file leaks the vocabulary under pressure — the model fills the gap with plausible-but-invented values.

### 4. Derived anti-criteria

Every forbidden action and non-ownership clause in the Agent Contract must yield at least one **code-asserted anti-criterion** — a test that fails if the forbidden thing happens. Prose forbiddance is not enforcement. Anti-criteria belong in Level 1 code assertions, never in an LLM judge.

### 5. Hard-to-vary acceptance criteria

A criterion is well-formed only if you can name the single probe (Read/Grep/Bash/curl/SELECT/test run) that returns yes/no on whether it is met. If you cannot name the falsifying test, it is not yet a criterion — it is a wish.

### 6. Conjecture/refutation learning trail on regressions

Each production failure regression entry carries four fields:
- **conjectured**: the belief that turned out wrong
- **refuted by**: the trace/observation that broke it
- **learned**: the corrected understanding
- **criterion now**: the new assertion added

An entry missing any field is a note, not a regression.

### 7. pass^k reliability metric

Track reliability with `pass^k` (does it succeed on ALL k attempts), not just `pass@1`. This exposes consistency that a single run hides — the metric that matters for anything autonomous.

### 8. The eval pyramid (Husain/Shankar)

- Level 1: Code assertions (every change, cheap)
- Level 2: LLM-as-judge (on cadence, **binary only**, calibrated against ≥100 human labels, TPR/TNR tracked)
- Level 3: Human review (~20-50 traces on major changes)

### 9. Tenant isolation as a principal dimension

`tenant_id` is derived from auth only, never from the model. Enforced below the LLM (row-level security / repository layer). Every path must be scoped: queries, tool calls, memory namespaces, cache keys, traces, sub-agent messages, background jobs. A code-asserted cross-tenant leakage eval runs in CI. The agent is a confused deputy — isolation enforced in the prompt will eventually leak.

### 10. MCP supply-chain controls

Community MCP servers are untrusted supply chain. Pin tool definitions by cryptographic hash; alert on any change. Version-pin and signature-check community servers; install only from an allow-listed registry. OAuth 2.1 + Resource Indicators; never pass tokens through.

### 11. The lethal trifecta (Willison)

An agent with (1) private data access, (2) untrusted content exposure, and (3) external communication ability becomes an exfiltration tool via prompt injection. Run this structural check on every deployment; if all three present, break one leg before shipping.

## Design decisions and trade-offs

### Prose + Operators split (ADR-0002)
Optimized for two audiences: humans who read and reason about the standard, and agents/editors who apply it in-session. Cost: the two must not drift — when the canon changes, both STANDARD.md and affected skills must update in the same change.

### Master router over monolithic skill (ADR-0001)
Progressive disclosure keeps context utilization low (the standard's own 40% rule). Cost: routing quality is a maintenance obligation — a stale description sends requests to the wrong sub-skill.

### Stability in canons, churn in vendors
The architectural canons (autonomy ladder, 5 patterns, single-vs-multi, harness) are deliberately stable. Vendor rankings and framework specifics are expected to churn — those PRs are the easy yes (GOVERNANCE.md).

### 2026 consensus: orchestrator-subagent, not peer-to-peer
Anthropic's research system, Claude Code Task tool, and Cognition's 2026 follow-up converge on a single lead spawning isolated subagents and consuming their summaries. Peer-to-peer agent buses, shared scratchpads, and free-form agent "debates" remain research curiosities — they multiply context, compound errors, and resist evaluation.

### M0-M3 maturity over fuzzy maturity
The SCORECARD.md is deliberately binary — a half-met control is a No. Your level is the highest band whose every gate item is satisfied. One unmet gate caps you at the level below. No partial credit, no skipping a band.

### Single-vs-multi-agent as context-engineering trade-off
Not a quality decision — a context-engineering one. Breadth-first parallelizable → multi-agent (isolated windows, ~15× tokens). Depth-first coherent → single-agent (shared context critical). The 2026 consensus has settled this question.

## The skill set architecture

Two tracks sharing the same sub-skills:

**agent-builder** (single-agent): Build, implement, review, or harden ONE production-grade agent. Bundles AGENT_STANDARD.md + copy-paste templates/. Triggered by "build an agent," "add tools to my agent," "review my agent code."

**agentic-product-architect** (multi-agent): Master router that classifies intent and dispatches to 11 sub-skills:
- architecture-design (autonomy ladder, 5 patterns, single vs multi)
- context-engineering (write/select/compress/isolate, 40% rule)
- harness-engineering (8-layer harness, cycle of trust)
- tool-design-mcp (MCP-first, <20 tools, RAG-MCP)
- memory-architecture (Mem0/Zep/Letta/LangMem/files/AgenticMind selection matrix)
- tenant-isolation (pooled/bridge/silo models, leakage paths, cross-tenant eval)
- durable-execution (Temporal Workflow+Activity pattern)
- eval-driven-dev (Husain/Shankar pyramid + judge calibration)
- framework-selection (constraint-based decision matrix)
- production-readiness (15-point Definition of Done audit)
- antipatterns-review (code review through 17 known failure modes)

Sub-skills are independently triggerable — asking "Mem0 or Zep?" loads memory-architecture directly.

## The templates (runnable artifacts, v2.0)

The v2.0 release added runnable security and CI artifacts, not just prose:

1. **lethal_trifecta_check.py** (70 lines) — CI gate that fails if an agent has private-data access AND untrusted-content exposure AND external egress with no mitigation declared
2. **injection_cases.yaml** — Indirect prompt-injection test cases (poisoned document/tool output), Promptfoo-style
3. **pin_mcp_tools.sh** (51 lines) — Hash-pins MCP tool definitions and detects rug pulls
4. **eval-gate.yml** — CI workflow that blocks merges when eval pass-rate drops below ≥90%

## Reference implementation: AgenticMind

A separate repo (github.com/Moai-Team-LLC/AgenticMind) that serves as the reference implementation for the memory/knowledge layer. Not an agent — an auditable, self-improving knowledge & memory substrate over MCP. The case study (examples/agenticmind-case-study.md) maps it clause-by-clause against the standard with file-level evidence. Gaps are marked as gaps with prioritized remediation.

## The Permission Tiers (P0-P6)

| Tier | Type | Approval |
|------|------|----------|
| P0 | Read | No |
| P1 | Draft | No |
| P2 | Internal Write | Usually no |
| P3 | External Write | Yes |
| P4 | Financial | Yes |
| P5 | Communication | Yes |
| P6 | Destructive | Always yes |

## The 15-point Definition of Done (v2.0, up from 12)

Context & state (3 items) + Tools & permissions (3 items + tenant isolation) + Reliability (3 items) + Evals & observability (3 items) + Security & identity (2 items) + Cost (1 item)

## Non-negotiable rules (22 in v2.0)

Key additions in v2.0: #19 (derived anti-criteria), #20 (closed enumerations), #21 (lethal trifecta), #22 (MCP tool pinning)

---

*Analysis based on full read of STANDARD.md (451 lines), AGENT_STANDARD.md (1597 lines), CONTEXT.md, SCORECARD.md, CHANGELOG.md, GOVERNANCE.md, ROADMAP.md, README.md, both ADRs, the case study, skill files, and templates.*
