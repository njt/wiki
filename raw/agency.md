---
title: "agency"
url: https://github.com/agentbureau/agency
date_fetched: 2026-05-14
section: "LLMs"
---

# Agency: AI Agent Composition Engine

Self-hosted system for building composable AI agents from reusable natural-language components called "primitives."

## Core Problem

Monolithic agent prompts become fragile and opaque. When one capability underperforms, developers must rewrite the entire prompt, risking breakage elsewhere. Agency makes agent components modular and individually improvable.

## Three Primitive Types

1. Role Components: Short capability descriptions (50-150 chars) defining what an agent can do
2. Desired Outcomes: Specifications of success criteria and output quality
3. Trade-off Configurations: Decision rules for conflicting values

## How It Works

Users request an agent through natural language conversation with Claude Code. Agency analyzes the task, selects matching primitives from its repository, and composes a specialized agent. The LLM dispatches the composed agent to execute the work. Performance data accumulates for learning which combinations work best.

## Technical Details

- Python 3.13+, installed via pipx
- Runs as background service exposing eight MCP tools
- Works with Claude Code, Superpowers, and Workgraph
- Stores primitives, compositions, and performance metrics separately
- Composes small agents for discrete tasks; not for long-running autonomous systems

## License

Code: Elastic License 2.0 (permits use and derivatives, prohibits commercial hosting)
Documentation: CC BY 4.0
