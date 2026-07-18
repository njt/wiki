# VulnHunter

Capital One's open-source agentic AI security tool that applies attacker-first reasoning to source code, forming a closed-loop Hunt → Fix → Verify pipeline. Built as a suite of Claude Code skills (pure prompt engineering) with Python helper packages for headless automation. Unlike traditional SAST scanners that flag suspicious patterns and produce false positives, VulnHunter traces forward from every attacker-accessible entry point, subjects each candidate to an adversarial falsification engine, and delivers evidence-backed remediations via test-driven development.

---

## Architecture

VulnHunter is five composable components organized around a three-phase closed loop:

```
┌──────────────┐     ┌─────────────────┐     ┌──────────────────────┐
│  /vulnhunt   │ ──▶ │ /vulnhunter-fix  │ ──▶ │ /vulnhunt-fix-verify │
│   (Hunt)     │     │     (Fix)        │     │      (Verify)        │
└──────────────┘     └─────────────────┘     └──────────────────────┘
        │                      │                        │
        ▼                      ▼                        ▼
┌──────────────────────────────────────────────────────────────┐
│              vulnhunter-agent (headless runtime)              │
│         Claude Agent SDK + GitHub automation + audit         │
└──────────────────────────────────────────────────────────────┘
        │
        ▼
┌──────────────────────────────────────────────────────────────┐
│              harness (batch + benchmark tooling)              │
└──────────────────────────────────────────────────────────────┘
```

### The Skills Layer

All three skills are **prompt-only** (pure Markdown instruction files) — no runtime code. They function as structured workflows that the orchestrator (Opus) follows deterministically.

**`/vulnhunt`** — The scan orchestrator (`vulnhunt/SKILL.md`, 353 lines + 10 phase files):

1. **Phase 1 (Recon)**: One subagent builds a complete input inventory — enumerating every external data entry point (HTTP params, gRPC fields, CLI args, queue messages, WebSocket payloads, etc.) — plus a subgraph partition table computed via union-find on a two-level application call graph
2. **Phase 2 (Hunt)**: Parallel trace agents dispatched by class group — 3 per subgraph partition (Injection/INJ, Navigation-Auth/NAV, Logic-Crypto/LOG) + 1 sink-driven auditor. Minimum count: `(3 × partition_count) + 1`. Each traces every input in its partition but only evaluates sinks for its class group. Dispatched in waves of max 6 to prevent API rate-limit saturation
3. **Phase 2b (Verify)**: Adversarial falsification — a single subagent explicitly tries to **disprove** every candidate. ~50% of candidates are historically false positives. Uses consensus-skip (if all partition agents cite the same elimination evidence, skip re-verification) and cross-subgraph visibility checks
4. **Phase 3 (Reproduce/Test/Fix)**: Exploit PoCs built for confirmed findings; Phase 3d sweeps for sibling instances using pattern matching
5. **Phase 4 (Report)**: Compiled from output files with strict anti-merge rules and instance-count cross-checks

Each candidate must pass **five hard gates** in order, with empirical source-code verification at each step: Gate 0 (designed behavior?), Gate 1 (reachability), Gate 2a (attacker-controlled?), Gate 2b (effective sanitization?), Gate 3 (new capability?).

**`/vulnhunter-fix`** — TDD-driven remediation (`vulnhunter-fix/SKILL.md`, 407 lines):
- Two modes: **in-place** (harvests vulnhunter-labeled GitHub issues, fixes on per-cluster git worktrees under `<repo>/.vulnhunter-fix/`) and **fork** (clones to private fork, delivers PRs there)
- Strict RED→GREEN pipeline: exploit demo → failing security test (with evidence persisted to disk as JSON) → fix → regression check → commit
- Phase-boundary operator checkpoints (no unattended phase transitions)
- Seven mechanical delivery gates before any `gh pr create`, including **anti-merge math**: a group of N findings is only allowed in one PR if `grouped_files ≤ 0.6 × split_files`
- Two-pass sweep algorithm: graph-anchored symbol pass (callers not routed through the fix = sibling defects) + regex pattern pass (CWE-specific patterns from `sweep-patterns.md`)
- 13 helper scripts, 12 reference files encoding fix patterns, crypto approved-lists, and severity rules

**`/vulnhunt-fix-verify`** — Read-only independent verification (`vulnhunt-fix-verify/SKILL.md`, 233 lines):
- Strict tool envelope: Read/Write/Edit/Glob/Grep/Agent — **no Bash, no network**
- Per-VULN verification through four gates, schema-validated output
- "Code is the source of truth" — developer comments are hints, not evidence

### The Headless Runtime

`vulnhunter-agent/agent/` (~3,400 lines of production Python) wraps the skills for unattended operation:

- **SDK integration**: Uses Claude Agent SDK (not CLI subprocess), giving direct access to streaming events, cost tracking, and session lifecycle
- **Cold-start retry**: Distinguishes 429/5xx before the first AssistantMessage (restart with backoff: 60s → 120s → 300s) from mid-stream transients (log and continue)
- **Stall detection**: 60 consecutive continuations with zero task lifecycle events = abort (catches genuinely hung subagents without killing rate-limited ones)
- **Dual-token GitHub auth**: Separate tokens for scan (clone + issue posting) and reports (publish push), preventing lateral movement if one is compromised
- **Audit stream**: JSONL emission of lifecycle events + per-finding observations for downstream ingest pipelines
- **Auth abstraction**: `ApiKeyTokenManager` and `OAuthTokenManager` share `get_valid_token()`, with Bedrock OAuth mode for enterprise proxy environments

## Key Techniques

### Forward-trace analysis with subgraph partitioning

VulnHunter inverts conventional SAST: instead of searching backward from dangerous sinks, it enumerates every entry point, builds an input inventory, and traces each input forward through the code. The subgraph partitioner (Phase 1d) uses union-find on a two-level call graph to group entry points into non-overlapping trace scopes, preventing agent context contamination. Shared infrastructure (modules imported by >50% of entry points) is factored out into a catalog that trace agents consult but don't scope to.

### Adversarial falsification as precision filter

Phase 2b is the precision engine. After parallel trace agents produce candidates, a single verification agent tries to disprove each one — re-checking gates, searching the full codebase for defenses invisible to partition-scoped agents, and applying comment skepticism rules (e.g., "by design" is not admissible as Gate 0 evidence; `sanitize()` naming is not proof of sanitization). The explicit claim that ~50% of candidates are false positives reflects a hard-won operational reality.

### TDD security fixes with mechanical delivery gates

The fixer applies test-driven development to vulnerability remediation: an exploit demo must exist on disk before any source file is edited. Seven mechanical gates (including anti-merge math, verification table completeness, and idempotency-key rules) run before PR creation. The sweep algorithm's two-pass design — graph-anchored symbol pass for precise caller tracking + regex fallback for coverage — catches sibling defects that a single-finding fix would miss.

### Context discipline in the orchestrator

The orchestrator SKILL.md explicitly forbids reading result files into context: "Do NOT read result files, recon output analysis, or source code into your context. Verify subagent completion by checking output files exist (Glob)." Subagent return messages are capped at ≤20 words. This keeps the orchestrator's context window free for coordination, not content — a deliberate engineering choice for long-running multi-agent workflows.

## Design Decisions

**Opus-only, by design**: The skills hard-gate on Opus 4.7+ and refuse to proceed on other models. The reasoning load (clustering, adversarial verification, fix synthesis) is calibrated for frontier-class reasoning. This trades broad compatibility for output reliability.

**Prompt-as-code, not prompt-as-conversation**: The phase files read like specification documents, not chatbot instructions. Dispatch rules are precise ("minimum agent count = (3 × production_partition_count) + 1"), not advisory. When a phase file is missing, the workflow stops with an installation error — no improvisation.

**Read-only by default**: Scans run without Bash — exploit tests are written but not executed. Code execution requires explicit `--enable-bash` + `--no-read-only` CLI flags, with Bash intentionally absent from the config-file allow-list so configuration drift can't silently re-enable arbitrary execution.

**Human gates between phases, not within them**: The fixer runs unattended within a phase but pauses at every phase boundary for operator approval. This creates clean audit checkpoints without interrupting the flow of individual tasks. The skill explicitly forbids "fast path" or "skip" options.

**Strict `git`/`gh` failure policy**: When sandbox TLS or keychain issues cause tool failures, the fixer stops and asks the user to run exactly one command in their own terminal. No retries, no alternative tools, no workarounds. This policy (documented in the SKILL.md with exact formatting templates) prevents the common agent failure mode of burning tokens on doomed retries.

## Comparison Notes

Unlike traditional SAST (Semgrep, CodeQL) which pattern-matches on syntax, VulnHunter reasons about **data flow provenance** — whether attacker-controlled data actually reaches a sink across trust boundaries, accounting for sanitization, type constraints, and framework protections.

Unlike [[Metis — ARM AI Security Code Review]], which combines tree-sitter call-graph reachability with LLM vulnerability confirmation, VulnHunter is pure LLM-driven — no static analysis engine. This gives it language flexibility at the cost of deterministic coverage.

Unlike [[OpenCodeReview]], which reviews code diffs in a hybrid deterministic+agent architecture, VulnHunter performs whole-codebase forward-trace analysis. The scope is larger, the token cost is higher, and the output is vulnerability reports rather than review comments.

The multi-agent orchestration parallels [[Orchestrating AI Code Review at Scale]] (Cloudflare's 7-agent + judge system) but inverts the pattern: Cloudflare fans out by review dimension, VulnHunter partitions by code subgraph then fans out by vulnerability class.

The fix pipeline shares DNA with [[no-mistakes]]'s AI-driven validation gates and [[Guardrails and Feedback Loops]]'s thesis that deterministic enforcement beats prompt-level pleading, but VulnHunter's gates are security-specific rather than general-purpose.

The read-only verifier's "code is the source of truth" discipline parallels [[Interdict]]'s PostgreSQL blunderbuss approach — validate against real state, not claims about state.

---

*Last updated: 2026-07-18*
*Tags: #tool #project #security #agents #code-review*

---

# Announcement Analysis (Capital One blog post)

## Key Quotes

> "After surfacing a finding, VulnHunter runs a structured reasoning workflow specifically designed to disprove its own argument."

This is the intellectual core of the tool and the part that separates it from every other AI security scanner. Most tools ask "is this vulnerable?" VulnHunter asks "why am I wrong about this being vulnerable?" — then discards the finding if it can answer that question. It's adversarial verification baked into the product rather than bolted on as a post-processing step. The same pattern appears in [[Guardrails and Feedback Loops]] (linters over prompts, deterministic enforcement) and [[The Advisor Strategy]] (second-pass judgment), but VulnHunter applies it to the hardest security problem: false positives are the reason developers ignore security tools.

> "Rather than the conventional sink-first approach that looks for dangerous code patterns in isolation, VulnHunter simulates an attacker's full journey."

The sink-first approach — grep for `eval()`, `system()`, `strcpy()` — is why SAST tools produce 90%+ false positive rates. A dangerous function in isolation tells you nothing about exploitability. VulnHunter's forward analysis from entry points through transformations to sinks is what a human penetration tester does. The question is whether an LLM can do it reliably enough to replace that human, or whether it produces a new category of false negatives: plausible-but-incomplete attack paths that look convincing but miss a subtle mitigation.

> "We built VulnHunter to give defenders a more rigorous, evidence-driven way to find and fix vulnerabilities before attackers can reach them."

The "before attackers can reach them" framing is important. Capital One isn't positioning this as a compliance tool or a checkbox in a CI pipeline. They're positioning it as a countermeasure to AI-accelerated attacks — the same technology that makes attackers faster should make defenders faster too. This is the arms-race framing that's been missing from most security tool announcements.

---

## Key Themes

#tool #security #agentic-ai #code-review #open-source #claude-code #vulnerability

**Falsification as first-class architecture.** Most AI tools treat verification as an afterthought — run the model, then maybe check the output. VulnHunter makes self-disproof a core architectural component. This is the right instinct, but the unexamined question is whether an LLM can effectively falsify its own reasoning. The same model that hallucinated an attack path is being asked to find the holes in that hallucination. There's a structural conflict of interest here that the article doesn't address.

**Attacker-first reasoning.** The forward-analysis approach (entry point → logic → checkpoint → sink) is philosophically correct. It's how real attackers think and how real pentesters work. But it's also computationally expensive — reasoning through every possible path from every entry point scales badly. The article mentions the tool ran across "thousands of repositories" at Capital One, which implies they solved the scaling problem somehow, but the mechanism isn't described.

**Claude Code skills as distribution.** Packaging a security tool as a Claude Code skill rather than a standalone binary or SaaS product is a bet on where the developer toolchain is heading. It's the same pattern as [[Load-Bearing Assumptions]] and [[Thought Refiner Skill]] — capabilities distributed as skills that compose with an existing agent harness rather than requiring their own runtime. The downside is vendor lock-in: VulnHunter currently requires Claude Opus 4.8 specifically. The article gestures at portability ("potential to be leveraged across coding harnesses and foundation models") but the current reality is single-model, single-harness.

**A bank shipping open-source security tooling.** Capital One is a regulated financial institution. The fact that they (a) built this internally, (b) ran it across their real codebase at scale, and (c) open-sourced it rather than keeping it as proprietary advantage — all of this is notable. The open-source rationale (interconnected supply chains mean collective defense) is genuine, but there's also a recruitment signal here: "we do interesting security engineering, come work here."

---

## Critical Analysis

**The falsification engine is brilliant in concept, unproven in practice.** The idea of having the tool try to disprove its own findings is the right architecture. But the article provides no data on what the falsification step actually catches — what percentage of initial findings are discarded? What kinds of false positives survive? An LLM asked to falsify its own reasoning is in a structurally awkward position; it's being asked to find flaws in the same cognitive process that produced the finding. There's a risk of a new failure mode: the falsification step fails to catch a false positive *and* adds a veneer of rigor that makes developers *more* likely to trust it. "It survived the falsification engine" could become the new "the linter passed."

**Compared to Metis, VulnHunter is all LLM, no static analysis.** [[Metis — ARM AI Security Code Review]] combines tree-sitter deterministic reachability analysis with LLM confirmation — the static analysis provides ground truth anchors. VulnHunter appears to be pure LLM reasoning with no deterministic component. This makes it language-agnostic (a strength) but also unmoored from any verifiable ground truth (a weakness). The falsification engine is supposed to compensate for this, but LLM self-critique is not the same as deterministic verification.

**The Claude Opus 4.8 dependency is both feature and limitation.** Optimizing for a specific model means the prompts and reasoning workflows can be tuned to that model's specific behavioral profile — it'll likely work better on Opus 4.8 than a model-agnostic tool would. But it also means the tool's effectiveness is tied to a single vendor's model availability and pricing. If you're a Capital One competitor running on AWS Bedrock with a different model, VulnHunter may not work well for you, despite the Apache 2.0 license.

**Supply chain defense is the right framing, but it's also self-serving.** Capital One's argument that "no single organization can solve this challenge alone" is correct — but it's also the standard playbook for open-sourcing internal tools: you get free community contributions and bug reports. The real test is whether Capital One continues to invest in the tool after open-sourcing it, or whether it becomes a dump-and-run.

**The "thousands of repositories" claim needs scrutiny.** Running a tool across your internal codebase and finding vulnerabilities is not the same as the tool being effective. Without a ground-truth benchmark (known vulnerabilities that the tool found vs. missed, false positive rates), the scale claim is marketing, not evidence. The article doesn't provide numbers on true positives, false positives, or false negatives.

---

## Connections

VulnHunter sits at the intersection of several threads in this wiki:

- **AI security scanning**: [[Metis — ARM AI Security Code Review]] (deterministic+LLM hybrid), [[OpenCodeReview]] (Alibaba's hybrid architecture), [[brooks-lint]] (pure prompt-engineering code review)
- **Security architecture**: [[Agentic AI Security Stack]] (unified threat model), [[Security and Sandboxing]] (containment patterns), [[How We Contain Claude]] (Anthropic's own security engineering)
- **Verification patterns**: [[Guardrails and Feedback Loops]] (linters beat prompts), [[Sherlock Agent Eval]] (adversarial verification improves accuracy)
- **Claude Code ecosystem**: [[Steering Claude Code]] (skills as infrastructure), [[Claude Code Mastery]] (practical Claude Code), [[Load-Bearing Assumptions]] (another Claude Code skill with falsification logic)
- **Supply chain**: [[Supply Chain Security for Software Developers]], [[Zero Trust for AI Agents]]
- **Code review landscape**: [[Agentic Code Review]], [[The End of Code Review]], [[Orchestrating AI Code Review at Scale]]

---
*Last updated: 2026-07-18*

*Sources: [[raw/vulnhunter]], [[raw/vulnhunter-repo]]*
