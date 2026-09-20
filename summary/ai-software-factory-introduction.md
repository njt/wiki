---
url: https://eriklieben.com/posts/ai-software-factory-introduction/
title: "An AI Software Factory — Introduction"
author: Erik Lieben
date_fetched: 2026-09-20
topics:
  - agent-orchestration
  - agent-coding-workflow
---

Erik Lieben opens a build-in-public series on "aifold", his self-hosted AI software factory written in .NET (ASP.NET Core + Aspire, with a Rider plugin and Angular web app). The framing: when you sit next to a coding agent you are simultaneously its trigger, sandbox, workflow, quality gate, budget, memory and reviewer — and a factory is what replaces each of those jobs with an explicit, checkable part. He explicitly rejects the "dark factory" ideal, grounding the design instead in Toyota's jidoka: automation does the work, checks stop the line, and people fix the cause, because software is custom work where checking *is* the process.

The parts list walks through twelve concerns: the brief as the entire input (with out-of-scope sections and acceptance criteria that exit zero); hand-off via an MCP server and a Claude Code skill that deliberately cannot start a run; gVisor-isolated sandboxes with per-run clones, egress allowlists and no model keys; pipelines of typed phases; gates that read evidence the agent didn't write; structural budgets; transcript-derived metrics including "cost per surviving kLOC"; layered person-confirmed memory; scoped skills; a runner-per-harness contract mixing Claude Code and Codex; a purpose-built review tool with typed comments; and agent fleets with a sanctioned message board.

It is offered as a learning exercise, not a recommendation — hosted agents may be the better deal — but the parts inventory is the point.
