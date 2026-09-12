---
title: "Agency — agentbureau/agency"
url: https://github.com/agentbureau/agency
author: Vaughn Tan (@arbois) / Agent Bureau
date_fetched: 2026-05-15
date_published: 2026-03-31 (v1.2.4.2)
topics:
  - misc
---

# Agency: AI Agent Composition, Assignment, and Evolution Engine

Self-hosted engine for composing AI agents dynamically from small, natural-language building blocks called "primitives." Instead of maintaining monolithic system prompts, users define discrete capabilities that can be mixed, matched, and improved independently. The system evaluates agent performance and uses that data to inform future compositions, creating a compounding improvement loop.

Stars: 34 | Language: Python 99.1% | License: Elastic 2.0 (code), CC BY 4.0 (docs) | 121 commits, 7 releases

## Core Problem

Monolithic agent prompts have no user-serviceable component parts. When one aspect of performance is weak, developers must rewrite the entire prompt, risking regressions in areas that already worked. Agency makes agent components modular and individually improvable.

## Three Primitive Types

1. **Role components** — individual capabilities an agent can possess (50–150 chars each). A single capability, typically one or two sentences (e.g., identifying gaps, grounding abstract arguments).
2. **Desired outcomes** — definitions of what quality output looks like, describing the shape and quality of good output (e.g., "Return a structured evaluation report with a numeric score").
3. **Trade-off configurations** — decision rules for when competing values conflict (e.g., preferring thoroughness over efficiency and flagging that choice in output).

All primitives are in natural language. A bundled primitive extractor can pull them from existing workflows, skills, and sessions.

## How It Works

Agency runs as a background service (`agency serve`). A user asks Claude Code to have Agency compose an agent for a specific task. Agency analyzes the request, performs fast semantic search over its primitive store, assembles a tailored agent, and returns the composition to the requesting LLM, which dispatches the agent to work.

The full protocol: assign → execute → evaluate.

## Real-World Case Study (v1.2.3 development)

An initial PRD review agent scored 26/60 — it read the spec in isolation but couldn't check against the actual codebase. After adding a one-sentence primitive for codebase-awareness (checking API signatures, SQL validity, testability), a re-run scored 55/60. The review caught a field name mismatch (`similarity` vs. `score`) that would have silently broken every relevance computation.

Across four review rounds, composed agents caught 14 implementation blockers that general-purpose Claude agents missed.

## Three Knowledge Layers

- **Primitives** — the building blocks themselves
- **Compositions** — which primitives were assembled for which tasks
- **Performance** — how each composition scored against specific criteria

This is "memory *about* agents, not memory *within* them." No individual agent carries context forward; the system does. Users can query patterns (e.g., which role components appeared in the five lowest-scoring review agents) and close gaps with a single primitive.

## MCP Tools (8 total)

| Tool | Purpose |
|------|---------|
| `agency_assign` | Compose agents for tasks, return rendered prompts |
| `agency_evaluator` | Provide evaluation criteria and callback JWT |
| `agency_submit_evaluation` | Submit structured evaluation results |
| `agency_get_task` | Retrieve task state, composition, evaluation status |
| `agency_list_projects` | Discover available projects |
| `agency_create_project` | Create new projects |
| `agency_status` | Check instance health, task progress, primitive counts |
| `agency_triage` | Lightweight primitive matching without full composition |

## What Agency Is Not

"Agency is not a runtime for long-running autonomous agents." It composes small agents for well-defined tasks and returns them to a requester (LLM, skill, pipeline, or human) for dispatch. It handles identity and composition; a separate runtime handles execution and persistence.

## Integrations

- **Claude Code** — MCP tools directly
- **Superpowers** — subagent dispatch (planner uses Agency to compose agents for each subtask)
- **Workgraph** — shell-based batch execution

Developed primarily for Claude Code. Compatibility with other LLMs not guaranteed.

## Installation

Requires Python 3.13+. `pipx install --python python3.13 agency-engine`, then `agency init` (setup wizard) and `agency serve`. Bundled `/agency-getting-started` skill for first-time users.

## License

Code: Elastic License 2.0 — allows use, copying, distribution, derivative works; prohibits providing as commercial hosted/managed service to third parties. Documentation: CC BY 4.0.
