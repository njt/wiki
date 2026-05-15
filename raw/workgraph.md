---
title: "workgraph"
url: https://github.com/graphwork/workgraph
date_fetched: 2026-05-14
section: "AI-enhanced Coding"
---

# Workgraph (wg)

A persistent task coordination system designed for human-AI organizations that centers work rather than agents. Records task dependencies, claims, evidence, failures, and human judgment in a durable, inspectable format.

## Core Philosophy
"Agents can come and go. The graph remains." The work graph is the primary organizational artifact, not the AI agents executing tasks.

## Key Features

**Persistent Infrastructure**
- Task graph stored as plain JSONL on disk (git-friendly, human-readable)
- Complete execution history with state transitions and logs
- Evidence and artifact tracking alongside tasks

**Operational Capabilities**
- Claims and handoffs allowing any agent (human or AI) to assume work
- Agent continuity through composable identities that outlive processes
- Human judgment points as first-class operations (verification, approval, rejection)
- Worktree isolation for parallel agents avoiding file conflicts

**What It's Not**
Not a chatbot, benchmark harness, project-management tool, or agent framework (LangGraph, CrewAI, AutoGen).

## Notable Claim
"The bottleneck is no longer only execution. It is knowing what was done, what failed, what evidence exists, where judgment entered."

## Proof of Concept
Poietic PBC was formed and organized entirely through wg, including company incorporation, grant drafting/submission, and research coordination.

Written in Rust (89.4%), supports multiple AI executors (Claude, Codex, OpenAI-compatible endpoints), TUI interface and CLI. MIT license.
