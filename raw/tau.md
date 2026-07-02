---
url: https://twotimespi.dev/
title: "Tau (τ): An educational coding agent you read like a textbook"
author: Alejandro Ao
date_fetched: 2026-07-03
date_published: 2025
---

# Tau (τ) — Educational Coding Agent

Tau is an open-source (MIT) terminal coding agent by Alejandro Ao (GitHub: alejandro-ao), published on PyPI as `tau-ai`. It's designed to be both a working tool and an educational resource — "a small Python coding agent you read like a textbook." The project is inspired by Pi (Mario Zechner's minimal coding agent) but is an original implementation.

## Three-Layer Architecture

1. **tau_ai** — Provider adapters translating OpenAI, Anthropic, Hugging Face, OpenRouter, and local endpoints into Tau's provider-neutral event stream.
2. **tau_agent** — The reusable harness: messages, tools, events, agent loop, session primitives. Provider- and UI-agnostic.
3. **tau_coding** — The coding environment: file/shell tools, sessions, skills, slash commands, and a Textual-based TUI.

Dependency direction: `tau_coding → tau_agent → tau_ai`.

## Design Principles

- **Small layers beat magic** — One job per package, each studyable independently.
- **Events make agents teachable** — The agent emits a typed event stream you can render, test, and export.
- **Real enough to matter** — Educational, not a toy. Runs as a real terminal agent.
- **Docs follow implementation** — Built phase by phase with notes on each addition.
- **Tools are ordinary typed functions** — Schema + async executor, no complex framework.
- **Sessions are durable and inspectable** — Append-only JSONL under `~/.tau/sessions/` with resume and branching.

## Features

- Interactive TUI (`tau`) and non-interactive print mode (`tau -p "prompt"`)
- Four built-in tools: read, write, edit, bash
- Slash commands: /login, /model, session management, compaction, export, theme switching
- Project instructions from AGENTS.md, .tau/, .agents/
- User skills and prompt templates
- Context accounting with manual/automatic compaction
- Multiple provider support normalized through tau_ai
- Programmatic use: import AgentHarness, iterate over events from harness.prompt()

## Educational Value

Tau demonstrates: provider-neutral streaming, agent loops with tool calling, typed local tools, durable sessions, session resume/branching, JSONL/HTML export, project instructions, skills, prompt templates, slash commands, context accounting, compaction, thinking controls, and UI adapter boundaries.

The project has 438 commits, primarily Python (93.7%), with documentation in Astro/HTML/JS/CSS. The core lesson: "Separate the brain, the environment, and the face."

## Repository

https://github.com/alejandro-ao/tau — MIT License, 438 commits.
