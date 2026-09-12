---
url: https://github.com/cloudflare/security-audit-skill
title: "Cloudflare Security Audit Skill"
author: Cloudflare
date_fetched: 2026-07-18
date_published: 2025
topics:
  - agent-orchestration
  - guardrails-and-feedback-loops
---

A Claude Code skill that orchestrates multiple parallel agents into a security
auditor, running a six-phase pipeline: recon, hunting, validation, reporting,
structured output, and independent verification. It is the open-source seed
from which Cloudflare's internal fleet-wide vulnerability harness grew.

The skill decomposes auditing into manageable chunks. Phase 1 launches three
parallel research agents to map architecture, trust boundaries, and input
surfaces. Phase 2 distributes attack classes across multiple hunter agents,
each scoped to one subsystem and armed with a 12-angle heuristic covering sad
paths, boundary conditions, implicit trust, operation ordering, concurrency,
parser differentials, and more. Core attack classes span injection, access
control, resource handling, cryptography, business logic, feature abuse,
chained attacks, a wildcard class for novel angles, and an "obvious things"
sweep. Four companion modules extend coverage to memory safety/binary targets,
LLM-backed applications, web protocols and auth, and client-side code.

The key innovation is adversarial validation. Findings must survive two
independent agents trying to disprove them before they reach the report.
Phase 3 launches fresh agents tasked explicitly with refuting each claim.
Phase 6 runs yet another set of independent verifiers that check every factual
assertion — file paths, line numbers, function names, trace chains, and
confidence scores — against actual source code. A strict JSON Schema with
oneOf-confirmed/rejected verdicts and a custom validator enforce this rigor.

The trade-off is explicit: optimized for low false positives over recall. A
single run catches roughly half of total vulnerabilities; multiple additive
passes are the intended workflow. The skill is agent-neutral in design but
assumes Claude Code's parallel sub-agent and filesystem capabilities in
practice. Limitations include no persistent learning between runs and
dependence on the underlying model's code-reading quality.
