---
url: https://www.uber.com/us/en/blog/ai-code-cleanup/
title: "Using AI to Delete Code: Agentic Tech Debt Cleanup at Uber"
author: Daniel Ramos, Diego Marcilio, Nikhil Parasaram, Yuxin Wang (Uber AI Verification / Programming Systems teams)
date_fetched: 2026-10-21
date_published: 2026 (recent, exact date not stated)
topics:
  - guardrails-and-feedback-loops
  - agent-architecture
---

Uber's AutoClean autonomously removes stale feature flags from production codebases. Once a flag is fully rolled out, the leftover conditional branches, parameters, config fields and tests become tech debt that humans deprioritize and that even AI agents pay for (they burn tokens parsing dead code). Cleanup looks like search-and-delete but unravels quickly: inlining a `true` forces propagation decisions through call sites, function signatures, config structs and test suites, some of which should be deleted and others rewritten.

The system has three stages. **Stage 1** is an agent with full codebase access but a deliberately scoped toolset — file search, code editing, and a boolean algebra/conditional simplification tool built on Uber's PolyglotPiranha code transformation engine, exposed over MCP. The design stance is explicit: agentic does not mean the agent does everything; transformations that can silently change program behavior stay deterministic. **Stage 2** lets CI be the source of truth. Fixers are tiered — cheap deterministic ones handle mechanical breakage (unused defaults, lint errors) and pass unrecognized failures to agentic fixers, each scoped so a test fixer can only edit test files (otherwise agents make failing tests pass by changing production code underneath them). **Stage 3** is validation beyond green CI: static checks for mechanically encodable defects, then "CodeCourt," a panel of property-reviewer agents run in parallel, each checking one property (no additions, semantics preserved, no intent drift, no agent artifacts). Any failure abandons the diff before a human sees it; humans review only business judgment ("should this flag go now?"), not mechanical correctness.

Results: over its lifetime AutoClean has cleaned up more than 16,000 flags and over 1,000,000 lines of code. Two transferable lessons stand out: single-agent end-to-end review lost to focused per-property subagents, and the last mile of autonomy is scoped permissions, not smarter models.
