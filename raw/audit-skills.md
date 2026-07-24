---
url: https://github.com/metacircu1ar/audit
title: Audit Skills for AI Coding Agents
author: Max Tikhomirov
date_fetched: 2026-07-25
date_published: 2026-06-06
---

# Audit Skills for AI Coding Agents

A library of 11 focused audit skills for AI coding agents, by Max Tikhomirov. Each skill is a small, self-contained playbook (42–77 lines) that an agent can run against a real codebase to report concrete findings with file paths, line numbers, impact, and suggested fixes.

## Repo Structure

The repo is a **degenerately flat skill library**: 11 self-contained `SKILL.md` files in individual directories under `skills/`, plus a README and LICENSE. No code, no runtime, no shared library, no dependencies. Each skill is a markdown prompt designed to be loaded by any AI agent's skill system (Claude Code, Gemini CLI, Codex, or any MCP-compatible harness).

```
skills/
  audit-input-validation/SKILL.md           (45 lines)
  audit-auth-and-access-control/SKILL.md    (56 lines)
  audit-secrets-and-config/SKILL.md         (42 lines)
  audit-rate-limiting/SKILL.md              (48 lines)
  audit-cors/SKILL.md                       (43 lines)
  audit-database-performance/SKILL.md       (44 lines)
  audit-resilience-and-observability/SKILL.md (45 lines)
  audit-asset-pipeline/SKILL.md             (42 lines)
  audit-type-safety/SKILL.md                (49 lines)
  security-review/SKILL.md                  (77 lines)
  ios-prelaunch-checklist/SKILL.md          (64 lines)
```

Total: 624 lines across 12 files. MIT licensed. Single commit by Max Tikhomirov on 2026-06-06.

## Skill Categories

The 11 skills fall into three functional groups:

| Group | Skills | Purpose |
|-------|--------|---------|
| **Code quality / correctness** | `audit-input-validation`, `audit-type-safety` | Find missing validation and type-system bypasses |
| **Security posture** | `audit-auth-and-access-control`, `audit-secrets-and-config`, `audit-rate-limiting`, `audit-cors` | Find auth gaps, leaked secrets, rate-limit holes, CORS misconfiguration |
| **Production readiness** | `audit-database-performance`, `audit-resilience-and-observability`, `audit-asset-pipeline`, `ios-prelaunch-checklist` | Find scaling risks, missing observability, asset delivery problems, iOS launch blockers |
| **Orchestration** | `security-review` | Run all 9 focused skills and synthesize into OWASP-style report |

## Skill Anatomy

Every skill follows an identical structure:

1. **YAML frontmatter** — `name` and `description` for registry integration
2. **Workflow** — numbered steps telling the agent what to find, check, and report
3. **Useful Searches** — grep-ready search terms specific to the audit domain (e.g., `route`, `router`, `controller`, `schema`, `validate` for input validation; `current_user`, `user_id`, `policy`, `authorize` for auth)
4. **Output format** — a structured template the agent must fill (tables for coverage audits, severity-grouped lists for findings)

## Key Design Decisions

**General over specific**: Skills are framework-agnostic and language-agnostic (except `audit-type-safety` which targets TypeScript). They rely on the agent's own codebase knowledge rather than hardcoding framework-specific patterns.

**Source-code analysis, not runtime testing**: Unlike the `Agent Skills for Security Testing` library which operates on mitmproxy traffic logs, these skills work against the source code itself. No runtime environment, test execution, or traffic capture needed — the agent reads code and reports findings from static analysis alone.

**Composite orchestration**: `security-review` is an umbrella skill that composes the 9 focused audit skills, maps results to OWASP risk classes (broken access control, injection, cryptographic failures, etc.), deduplicates findings, and validates high-impact claims by re-reading code paths. This is the same composition pattern seen in Cloudflare's security audit skill and PAAD's specialist+verifier pattern.

**Output as reliability mechanism**: Every skill mandates finding format (file:line + impact + fix + missing test). This isn't cosmetic — it forces the agent to produce verifiable, reproducible findings rather than vague claims. A finding without a file reference isn't a finding; a fix that doesn't match the codebase's style isn't a fix.

**No shared protocol between skills**: Skills are intentionally independent — no shared state, no cross-skill deduplication, no library. Any skill works alone. The cost is that running all 11 sequentially costs 11× the context. The `security-review` umbrella handles deduplication and synthesis.

## Comparison to Related Libraries

- **vs. Agent Skills for Security Testing (instavm)**: Both are flat skill libraries with YAML + markdown prompts. Security-testing uses mitmproxy traffic as input; audit-skills works against source code. Security-testing has the 4,000-HackerOne-report moat; audit-skills encodes general engineering wisdom. Complementary: runtime pentesting vs. static production-readiness audit.
- **vs. Cloudflare Security Audit Skill**: Cloudflare's is a multi-phase parallel-agent pipeline for exploit hunting; audit-skills are single-agent playbooks for broad coverage. Cloudflare targets deep vulnerability discovery; audit-skills targets breadth — the "did you forget anything?" launch checklist.
- **vs. PAAD (Ovid)**: Both are Claude Code skill suites with composable skills. PAAD targets the development *workflow* (spec critique, architecture review, TDD guard); audit-skills targets the *codebase* (validation, auth, secrets, performance). PAAD's pushback → alignment → architecture → review pipeline is a workflow guard; audit-skills' input-validation → auth → secrets → rate-limiting is a surface-area audit.
- **vs. Agent Skills Library (dzhng)**: dzhng's library is a software factory operating system (plan → slice → build → verify); audit-skills is a QA/audit toolkit. dzhng builds; audit-skills checks. Both share the "skills as composable units" philosophy and harness-agnostic stance.

## Architecture Analysis

The architecture is **degenerately flat** — the simplest possible structure that achieves the goal. No shared code, no plugin system, no registry integration, no versioning machinery. Each skill is a standalone markdown file. This is not laziness; it's a deliberate stance that the skill format (YAML frontmatter + markdown instructions) is sufficient and more infrastructure would reduce portability.

The composition strategy is **LLM-mediated**: rather than a deterministic orchestrator that calls skills programmatically, the `security-review` skill instructs an agent to "run focused sibling skills when applicable." Deduplication, cross-referencing, and synthesis are all left to the agent's reasoning — the skills provide the *what* and *how to find*, not the *how to combine*.

This is a bet on agent capability: it assumes the agent can independently execute each skill, hold findings in context, deduplicate across skill outputs, and synthesize a coherent report. For modern frontier models this is reasonable; for weaker models or when context windows overflow, quality degrades.

The project's main weakness is the absence of evaluation. There are no example audit reports, no benchmark of findings-per-skill, no regression suite that verifies skills don't drift. A skill library without evaluation is like a test suite without assertions — you trust but cannot verify. Compare to [[Razorback]]'s reproducible benchmarking or [[FrontierCode]]'s Diamond/Platinum/Gold tiering. This is less a criticism of metacircu1ar and more a reflection of where the skill-library ecosystem is: evaluation infrastructure doesn't exist yet.

The project's main strength is its **curatorial discipline**. Eleven skills, 624 lines total, each scoped to one audit dimension. No feature creep, no "20 skills for every possible framework," no tutorial padding. The skills are operational — they tell the agent what to grep for, what to check, what to report — rather than educational. This is the same curatorial taste that distinguishes [[Agent Skills Library (dzhng)]]: fewer invocation points over more granularity.

## Key Techniques

### Parameter/pattern vocabularies as search instruction

Rather than teaching the agent what an auth bypass *is* (the model already knows), each skill teaches the agent **what to grep for**. The auth skill catalogs risky search terms: `current_user`, `user_id`, `account_id`, `role`, `admin`, `policy`, `authorize`, `find`, `get`, `delete` — operational knowledge about where vulnerabilities hide in a codebase.

### Route-to-form mapping

The input validation skill instructs the agent to build a **route-to-form map** by following client API calls to server handlers. This is the same gRPC/REST endpoint inventory pattern from [[How AI Coding Agents Actually Use Your Technology]], applied to audit rather than API design. The agent traces every user-controlled input from UI through to server handler, building a coverage matrix.

### Cross-layer mandatory server validation

A recurring theme across skills: treat server-side validation as mandatory even when client validation exists. TypeScript types, UI placeholders, HTML input types, and OpenAPI documentation do not count unless runtime code enforces them. This is a concrete defense against the "I validated it on the client" anti-pattern that frameworks encourage.

### OWASP-style risk classification for composition

The `security-review` skill maps findings from all 9 focused skills onto standard OWASP risk classes (broken access control, injection, cryptographic failures, security misconfiguration, etc.). This is a **standardization bridge** — it translates the library's internal finding format into the risk taxonomy that security teams and compliance frameworks speak. The same approach appears in [[Metis — ARM AI Security Code Review]]'s SARIF-native output.

### Blast-radius prioritization

Multiple skills use blast radius as a prioritization heuristic: shared utilities rank above isolated UI glue, auth/payment code ranks above documentation, `any` in a type exported from `utils.ts` ranks above `any` in a one-off component. This is the same differential-priority approach from [[Building Agents for Production Systems with MCP]] and [[Steering Claude Code]] — focus the agent's attention where the cost of being wrong is highest.

### Framework-agnostic search guidance

Instead of hardcoding `grep -r "@PostMapping"` or `rg "app.get("`, skills provide semantic search terms (`route`, `router`, `controller`, `handler`) and instruct the agent to "use the repository's framework conventions first." This makes skills portable across stacks but adds a step: the agent must discover the framework's routing pattern before it can audit.

## Context Budgeting

The flat architecture is fundamentally a context-window optimization. Each skill is 42–77 lines — small enough that a frontier model can load the skill, hold it in context, read relevant code files, and produce findings within a single context window. The trade-off is that comprehensive coverage requires running multiple skills sequentially, each in its own context window, and the agent must carry findings forward. Compare to [[Cloudflare Security Audit Skill]]'s parallel-agent approach where all dimensions run simultaneously; meta-circu1ar's library is sequential by design but more portable.
