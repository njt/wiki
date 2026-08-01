---
url: https://github.com/metacircu1ar/audit
title: "Audit Skills for AI Coding Agents"
author: Max Tikhomirov
date_fetched: 2026-07-25
date_published: 2026-06-06
---

A library of 11 focused audit skills for AI coding agents, each a self-contained markdown playbook (42–77 lines) that an agent runs against a real codebase to produce concrete findings with file paths, line numbers, impact, and suggested fixes.

The skills fall into three groups: code quality (input validation, type safety), security posture (auth, secrets, rate limiting, CORS), and production readiness (database performance, resilience, asset pipeline, iOS prelaunch). A tenth security-review skill composes all nine focused audits into an OWASP-style report, deduplicating and cross-referencing findings across dimensions.

The architecture is deliberately flat — each skill is a standalone markdown file with no shared code, runtime, or dependencies. Skills are framework-agnostic and operate on source code via static analysis rather than runtime testing. They teach the agent *what to grep for* (parameter and pattern vocabularies) rather than explaining security concepts the model already knows.

Every skill mandates a structured output format (file:line, impact, fix, missing test), which forces verifiable, reproducible findings. A finding without a file reference is treated as not being a finding at all. The composite security-review skill translates internal findings into standard OWASP risk classes, bridging to the taxonomy that security teams and compliance frameworks expect.

Compared to related libraries: unlike Agent Skills for Security Testing (which works from mitmproxy traffic), these skills audit source code directly. Unlike Cloudflare's parallel-agent pipeline for deep exploit hunting, these are single-agent playbooks optimized for breadth — a "did you forget anything?" launch checklist. The project's main strength is curatorial discipline: 11 skills, 624 lines total, no feature creep. Its main gap is the absence of evaluation — no example reports, benchmarks, or regression suite to verify skill quality over time.
