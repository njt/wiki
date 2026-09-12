---
url: https://github.com/ovid/paad
title: "PAAD — Defense-in-Depth for AI-Assisted Development"
author: Curtis "Ovid" Poe
date_fetched: 2026-07-25
date_published: 2026-03-14
topics:
  - guardrails-and-feedback-loops
  - agent-coding-workflow
---

PAAD is a Claude Code plugin marketplace and multi-platform skill suite that
adds defense-in-depth safeguards to AI-assisted development. It catches failure
modes AI coding assistants don't reliably catch on their own: weak specs,
plan-spec drift, architectural decay, and skipped quality gates. It supports
Claude Code natively and ships compatible skill files for Cursor, Kiro, and
Antigravity.

Eight skills form the suite. **Pushback** critiques specs before implementation
begins, checking for contradictions, feasibility, scope problems, omissions,
ambiguity, and security concerns — one issue at a time to respect developer
attention. **Alignment** checks that requirements, plans, and tasks are
consistent, then rewrites all action items in TDD format with specific
red/green/refactor steps. **Agentic-architecture** dispatches five specialist
agents in parallel, each examining the codebase through a distinct architectural
lens across 34 flaw types and 14 strength categories. **Fix-architecture**
guides iterative, sequential fixing of those flaws with mandatory safety-net
tests before any refactoring begins. **Agentic-review** is a pre-merge quality
gate using five bug-hunting specialists covering logic, error handling, contract
compliance, concurrency, and security. **Agentic-a11y** audits accessibility
across eight platform types with platform-specific checks per specialist,
targeting WCAG 2.2 AA. **Vibe** guards small fixes with mandatory TDD
pre-flight checks. **Makefile** generates and updates project Makefiles with
strict rules against modifying existing targets without approval.

The core architectural pattern across the multi-agent skills is
specialists-then-verifier: dispatch parallel agents with distinct lenses, then
have a single skeptical verifier read actual code at every referenced file:line,
drop false positives, deduplicate findings, and assign severity. This treats the
LLM's tendency to generate plausible-but-wrong claims as a solvable
reliability problem rather than a given.

Every skill except help includes a Graphviz decision digraph encoding its safety
gates visually, making it harder for the LLM to skip pre-flight checks. The
build system enforces this and five other invariants via Makefile targets.
Skills are invoked as `/paad:<name>` in Claude Code and via natural language on
other platforms. A conversion script strips Claude Code-specific paths and
references for the Kiro and Antigravity ports.

The philosophy is honest about risk, keeps humans in the loop for every
consequential decision, and optimizes for better outcomes over minimum token
consumption.
