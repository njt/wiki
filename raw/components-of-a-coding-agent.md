---
title: "Components of a Coding Agent"
url: https://magazine.sebastianraschka.com/p/components-of-a-coding-agent
author: "Sebastian Raschka, PhD"
date_fetched: 2026-05-15
date_published: 2026-04-04
---

# Components of A Coding Agent - Sebastian Raschka

## Overview
Raschka examines how coding agents (Claude Code, Codex CLI) wrap LLMs in an "agentic harness" to improve real-world coding performance, arguing the surrounding system often matters as much as the model itself.

## Taxonomy

**LLM:** "the core next-token model"
**Reasoning model:** An LLM trained or prompted to "spend more inference-time compute on intermediate reasoning"
**Agent:** "a control loop around the model" that decides what to inspect, which tools to call, how to update state, and when to stop
**Agent harness:** "the software scaffold around an agent that manages context, tool use, prompts, state, and control flow"
**Coding harness:** A task-specific harness for software engineering

Key claim: "vanilla versions of LLMs nowadays have very similar capabilities," so the harness "can often be the distinguishing factor that makes one LLM work better than another."

## Six Core Components

1. **Live Repo Context** — Agents gather workspace facts upfront (git branch, status, commits, AGENTS.md, README) so they don't "start from zero, without context, on every prompt."

2. **Prompt Shape and Cache Reuse** — Smart runtimes maintain a "stable prompt prefix" (instructions, tool descriptions, workspace summary) that changes infrequently, while only the user request, recent transcript, and short-term memory update each turn.

3. **Structured Tools, Validation, and Permissions** — The harness provides "a pre-defined list of allowed and named tools with clear inputs and clear boundaries." Tool calls go through validation checks (known tool? valid arguments? user approval needed? path within workspace?) before execution. "The harness is giving the model less freedom, but it also improves the usability at the same time."

4. **Minimizing Context Bloat** — Coding agents face worse bloat than regular chats due to "repeated file reads, lengthy tool outputs, logs, etc." Three strategies: Clipping (shortens overlong outputs), Transcript reduction/summarization (recent events stay richer; older ones compress more aggressively), Deduplication (removes repeated file reads). Raschka calls this "one of the underrated, boring parts of good coding-agent design," adding that "a lot of apparent 'model quality' is really context quality."

5. **Structured Session Memory** — Two-layer storage: Full transcript (durable record of all requests, outputs, and responses; stored as JSON, resumable) and Working memory (distilled version with current task, important files, and recent notes). The compact transcript is for "prompt reconstruction," while working memory handles "task continuity."

6. **Delegation with Bounded Subagents** — Subagents allow parallelization of subtasks. The design challenge is binding them — inheriting enough context to be useful while imposing constraints like read-only access or restricted recursion depth. "The tricky design problem is not just how to spawn a subagent but also how to bind one."

## Comparison to OpenClaw

OpenClaw is "more like a local, general agent platform that can also code" rather than a specialized terminal coding assistant. Coding agents are "optimized for a person working in a repository," whereas OpenClaw handles "many long-lived local agents across chats, channels, and workspaces."

## Notable Quotes

"A good coding harness can make a reasoning and a non-reasoning model feel much stronger than it does in a plain chat box"

"Coding work is only partly about next-token generation. A lot of it is about repo navigation, search, function lookup, diff application, test execution, error inspection"

"The agent is the system that repeatedly calls the model inside an environment"

On the harness vs. model debate: if you dropped a capable open-weight LLM "into a similar harness, it could likely perform on par with" leading models

## References

- **Mini Coding Agent:** github.com/rasbt/mini-coding-agent (pure Python implementation demonstrating all six components)
- **Build a Large Language Model (From Scratch)** — Raschka's earlier book
- **Build a Reasoning Model (From Scratch)** — Newly completed, 528 pages in early access on Manning; covers evaluation, inference-time scaling, self-refinement, reinforcement learning, and distillation
