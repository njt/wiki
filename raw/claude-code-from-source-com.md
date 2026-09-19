---
url: https://claude-code-from-source.com/
date_fetched: 2026-09-19
---

# 
How Anthropic built the most widely used

 AI coding agent

 When Claude Code shipped on npm, the source maps came with it. We read every file. This book distills the architecture, design decisions, and transferable patterns into 18 chapters you can learn from and apply to your own systems.

Start reading## What you'll learn

### The agent loop

How an async generator drives the entire system — streaming model output, executing tools, recovering from errors, and compressing context across 4 layers.

### Tool execution at scale

A 14-step pipeline from model request to tool result. Permission resolution, speculative execution, concurrent batching by safety classification.

### Multi-agent orchestration

How sub-agents share prompt cache prefixes to cut costs by 95%. Fork agents, coordinator mode, swarm teams with mailbox messaging.

### Memory without a database

File-based memory with an LLM-powered recall system. Four memory types, staleness warnings, and a Sonnet side-query that beats embedding search.

### Performance engineering

Startup in 240ms via parallel I/O. Slot reservation saving context in 99% of requests. Bitmap pre-filters for fuzzy search. Every millisecond accounted for.

### Extensibility and security

Two-phase skill loading (metadata at startup, content on demand). 27 lifecycle hooks with config snapshots frozen at startup to prevent injection.

## Explore the architecture

Six core abstractions power Claude Code. Drag nodes to rearrange, hover for details, click to read the chapter.

## Who this is for

**Engineers building agentic systems.** Every chapter ends with "Apply This" — 5 transferable patterns with concrete adaptation advice. Steal the architecture, skip the mistakes.

**Technical leaders evaluating architectures.** Follow the narrative without reading every code block. Understand why decisions were made, not just what was built.

**Anyone curious about how production AI tools work.** Claude Code is used by hundreds of thousands of developers. This is how it works under the hood.

## Table of contents

### Foundations

Before the agent can think, the process must exist.

The 6 key abstractions, data flow, permission system, build system

5-phase init, module-level I/O parallelism, trust boundary

Bootstrap singleton, AppState store, sticky latches, cost tracking

Multi-provider client, prompt cache, streaming, error recovery

### The Core Loop

The heartbeat of the agent: stream, act, observe, repeat.

### Multi-Agent Orchestration

One agent is powerful. Many agents working together are transformative.

### Persistence and Intelligence

An agent without memory makes the same mistakes forever.

### The Interface

Everything the user sees passes through this layer.

### Connectivity

The agent reaches beyond localhost.

### Performance Engineering

Making it all fast enough that humans don't notice the machinery.

## How this book was made

The source was extracted from npm source maps — the `.js.map` files that shipped with Claude Code contained a `sourcesContent` field with the full original TypeScript. Nearly two thousand files comprising the complete architecture.

 **36 AI agents** analyzed and wrote the entire book in four phases:

The entire process — from source extraction to final revised book — took approximately **6 hours**. A final audit pass ensured no verbatim source code remained — every code block was rewritten as pseudocode with different variable names.

## The 10 patterns that make it work

If you read nothing else, these are the architectural bets that define Claude Code.

 **Purely educational.** This book contains no source code from Claude Code — every code block is original pseudocode written to illustrate architectural patterns. The goal is to help engineers understand how production AI agents are built, not to reproduce proprietary software. The "NO'REILLY" cover is a parody/meme for illustrative purposes only — no affiliation with O'Reilly Media.
