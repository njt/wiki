---
url: https://8thlight.com/insights/teaching-the-agent-our-craft-structured-agentic-development-on-a-real-codebase
title: "Teaching the Agent Our Craft: Structured Agentic Development on a Real Codebase"
author: Alex Haldeman
date_fetched: 2026-07-18
date_published: 2026-07-06
---

Alex Haldeman, Lead Engineer at 8th Light, describes how his team built a digital
chronic-pain-treatment platform using Claude Code with a structured, disciplined
workflow. The article is a case study in making agentic development repeatable
rather than ad-hoc.

The team adapted Tyler Burleigh's Research-Plan-Implement (RPI) model, injecting
explicit review cycles between phases so product managers and designers stayed
involved at each transition. The framework lives entirely in a `CLAUDE.md`,
`.mcp.json`, and `.claude/` directory — no application code.

**CLAUDE.md** serves as the knowledge root: project context, conventions, and
four critical rules (no speculative code, tests before implementation, reuse
before new, comments explain why not what). **Path-scoped rules** load
automatically based on which files the agent touches, encoding team conventions
like Testing Library selector patterns and FastAPI dependency-wiring structure
that developers normally absorb only after several code reviews.

**MCP integration** wired Linear (backlog) and Figma (design system) directly
into the agent's context. Developers pulled design tokens and UI flows without
copy-pasting; re-syncing a built component against an updated design became a
single command.

Two custom skills orchestrate the work. **`/story-writer`** handles research and
planning: it reads (or creates) a Linear ticket, explores the codebase to
ground implementation steps in real paths, and conducts an interview loop to
sharpen requirements into acceptance criteria. **`/tdd-build`** reads the
ticket, proposes numbered TDD cycles (each with test file, behavioral spec,
command, RED gate reason, and target files), and dispatches a `test-writer`
subagent for each cycle. The subagent starts from a fresh context with no
knowledge of the implementation, writes a failing test, and confirms the RED
gate before the main agent writes the minimum code to pass. A PreToolUse hook
enforces that the test-writer only touches test directories — the prompt states
the intent, the hook enforces it.

The key result: two developers carried what would normally require a larger
team — a multi-tenant platform, an AI coaching companion under safety
constraints, and a privacy-conscious data architecture. Haldeman is explicit
that the harness was "an accelerant to our team's abilities, not a replacement."
Engineers stepped in when the agent needed a cleaner structure, then folded
those lessons back into the rules. That feedback loop — engineers improving the
harness and the harness letting them cover more ground — is what made the
approach stick beyond experimentation.
