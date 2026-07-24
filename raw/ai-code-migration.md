---
url: https://claude.com/blog/ai-code-migration
title: "How Anthropic runs large-scale code migrations with Claude Code"
author: Michael Segner (based on migrations run by Jarred Sumner and engineering teams across Anthropic)
date_fetched: 2026-07-25
date_published: 2026-07-16
---

# How Anthropic runs large-scale code migrations with Claude Code

**Author:** Michael Segner, based on migrations run by Jarred Sumner and engineering teams across Anthropic
**Published:** July 16, 2026
**Reading time:** 5 min

---

The central insight: "You fix the process (loop) that produced the code" rather than fixing the code itself.

---

## Two Key Migration Examples

### 1. Bun (Zig → Rust) — Jarred Sumner

- Co-founder of Bun, Member of Technical Staff at Anthropic
- ~1 million lines of code produced in under two weeks
- "100% of Bun's existing test suite passing in CI before merge"
- 19 regressions post-merge, all since fixed
- Consumed 5.9 billion uncached input tokens and 690 million output tokens — approximately **$165,000 at API pricing**

### 2. Python → TypeScript — Mike Krieger

- Co-lead of Anthropic Labs
- Migrated a Python codebase to **165,000 lines of TypeScript over a weekend**
- Included hundreds of agents, eight phase gates, three adversarial review rounds, and a parity check
- Main portion was 27 million tokens
- Motivation: Python toolchain took ~8 min per platform to compile; Rust port compiles in ~2 seconds, binary starts 6x faster

---

## Why AI Changes the Math for Code Migrations

Five reasons cited:
1. **The work is parallel** — files/crates can be migrated simultaneously
2. **Context is clear and comprehensive** — old code serves as a spec
3. **Built-in referee** — existing test suites verify work objectively
4. **The queue writes itself** — compiler/test failures become the next task
5. **Consistency and edge case handling** — drift has nowhere to hide; edge-case fixes become rules all agents follow

---

## Six-Step Process

### Prerequisites: Build a "Judge"

- Categorize existing tests (identify which are expressible as external calls)
- Rewrite tests for portability across both codebases
- Validate: confirm it passes on original code and fails on deliberately broken code
- Mike created a parity harness of 7 real-world scenarios

### Step 1 — Create the Rulebook, Dependency Map, and Gap Inventory

- **Rulebook:** Translate types/idioms between languages; built by chatting with Claude
- **Dependency map:** Claude Code deploys agents to map file dependencies (critical for parallel workstreams)
- **Gap inventory:** Documents where the new language requires things the old one didn't (e.g., Rust's memory management vs Zig's; TypeScript's interfaces vs Python's duck typing)
- Used 8 subagents to review for different failure mode categories

### Step 2 — Stress-Test the Rules

- Mini-migration "shakedown cruise" — translate a handful of files multiple ways and compare
- Jarred had one agent translate with the rulebook, one "like a senior Rust engineer," and one to derive new rules from the diff
- "Throw out any translated files. The goal is to refine the rules, not make incremental progress"

### Step 3 — Translate Everything

- Multi-agent loop: implement → review → fix
- Smaller models (Claude Sonnet) used for high-volume implementation fan-out; largest models reserved for reviewers/rule-writers
- Unresolvable items flagged with `// TODO(port): <reason>`
- Two adversarial reviewers evaluate work; disagreements go to a third agent
- Systemic issues: "add one sentence to the rulebook and regenerate the affected batch"

### Steps 4, 5, 6 — Compile, Run, and Match Behavior

- Shared loop architecture requiring progressively less human judgment
- **Step 4 (Compile):** Orchestrator script invokes compiler; fixer agents work through error list in parallel
- **Step 5 (Run):** Smoke tests find crashes; issues grouped by root cause
- **Step 6 (Match behavior):** Run test suite against both codebases; fixer agents review failures; a build daemon serializes the expensive rebuild step
- Mike's variant: small script running 7 real-world scenarios diffed against original; Claude then designed its own end-to-end test suite and ran it autonomously overnight for 4 nights

---

## Best Practices

1. **Don't follow the guide blindly** — treat it as a starting point and plan your specific migration with Claude first
2. **Don't focus on individual failures** — that's the loop's job; human attention belongs on patterns
3. "Make review adversarial and verification mechanical" — let scripts be the referee
4. "Don't use the largest model for everything" — smaller models handle high-volume implementation; save largest for reviewers and rule-writers
5. **Front-load the human hours** — rulebook and stress test are the most time-consuming parts
6. **Make the work queue mechanical and resumable** — "done" means the output file exists on disk

---

## Results of the Bun Migration (Now in Production)

- ~4% of Rust code sits inside "unsafe" blocks (mostly single-line pointer operations at C/C++ boundaries)
- Memory in one benchmark: dropped from **6,745 MB to 609 MB**
- Binary is **19% smaller** on Linux and Windows
- **2–5% faster** across HTTP serving and real-world workloads (next build, tsc)

---

## Related Resources

- Migration starter kit: https://github.com/anthropics/code-migration-kit-with-claude-code
- Code-modernization plugin: https://github.com/anthropics/claude-plugins-official/tree/main/plugins/code-modernization
- Dynamic workflows in Claude Code: https://claude.com/blog/introducing-dynamic-workflows-in-claude-code
