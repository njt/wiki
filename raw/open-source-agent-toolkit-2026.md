---
url: https://www.oreilly.com/radar/the-open-source-agent-toolkit-in-2026/
title: "The Open Source Agent Toolkit in 2026"
author: Paolo Perrone
date_fetched: 2026-07-18
date_published: 2026-07-14
---

# The Open Source Agent Toolkit in 2026

**Author:** Paolo Perrone
**Published:** July 14, 2026 • 17 minute read
**Source:** Originally appeared on Paolo Perrone's Substack, *The AI Engineer*, republished on O'Reilly Radar with permission.

---

## Overview

The article examines the 2026 open source AI agent ecosystem across seven architectural layers, noting that while most production problems have been solved, they've been "solved each one in a dozen incompatible ways." The key to picking tools is identifying which constraint—latency budget, audit trail, model portability, or language stack—dominates for your system.

The framing is that these layers are seven independent decisions, not a vertical stack where one choice constrains the next. The teams shipping reliable agents are those who "picked the best tool per layer and accepted that integrating the seams is part of the job."

---

## Layer 1: Orchestration and Runtime Control

This layer manages the think-act-observe loop.

**LangGraph** is described as the Python production default—a graph-based state machine with durable execution via PostgresSaver, time-travel debugging, and major enterprise adopters. However, it's verbose; a simple sequential tool call requires defining state schemas, nodes, edges, and compilation.

**CrewAI** offers the lowest setup overhead, letting you declare roles (researcher, writer, reviewer) without defining a state schema. The trade-off: it "optimizes for prototype velocity at the cost of production durability," lacking resume-from-crash capability and per-node error handling.

**Pydantic AI** treats every agent output as a typed Pydantic model, with validation and retries built in. It's best for single-loop agents returning validated data to downstream services but has weaker multi-agent primitives.

**Mastra** is the TypeScript-native option from ex-Gatsby founders, designed to integrate into Next.js apps without a Python sidecar. It has a smaller ecosystem and fewer production case studies than LangGraph.

Vendor SDKs (Claude Agent SDK, OpenAI Agents SDK, Google ADK) remove orchestration friction but lock the agent to one provider's API.

---

## Layer 2: Memory and State

The article makes a critical distinction: "The context window isn't memory." Production agents keep memory in a dedicated layer outside the prompt.

**Mem0** offers user-scoped, session-scoped, or agent-scoped memory with hybrid storage combining vectors and a graph. It has mature SDKs integrating with LangGraph, CrewAI, and Mastra, with 48,000+ GitHub stars. The article cites a benchmark showing 92% lower latency and 93% fewer tokens versus naive full-context approaches. Its limitation: it treats memory as retrieval, lacking temporal reasoning about how facts and relationships change over time.

**Zep/Graphiti** is the temporal graph option, performing entity resolution and tracking how relationships evolve. This enables queries like what a customer's status was in Q2 or when a contract owner changed. The trade-off is that graph construction is expensive—memory footprint per conversation runs past 600,000 tokens versus Mem0's 1,764, with delayed postingestion retrieval. Choose Zep when agents need historical reasoning and can tolerate seconds between turns.

**Letta (formerly MemGPT)** treats memory as an operating system does: main context as RAM, archival storage as disk, with the agent deciding what to promote or archive. It's fully open source, model agnostic, and self-hosted. The trade-off: harder to deploy than a hosted Mem0 endpoint and more difficult to debug since memory decisions happen inside the agent at runtime.

An engineering lesson distinguishes **runtime state** (the agent's scratchpad mid-task—LangGraph's PostgresSaver handles this) from **knowledge memory** (what the agent learned across sessions—Mem0 and Zep handle this). Conflating them produces agents that either crash without recovery or forget users between sessions.

---

## Layer 3: Protocols and Tools

The article states that in 2026, this layer is MCP (Model Context Protocol)—the open standard used by Claude Agent SDK, OpenAI Agents SDK, Google ADK, and every serious framework. The orchestration choice from layer 1 already determines MCP integration.

**FastMCP** is the Python framework for writing MCP servers, decorator-based and async-first. **mcp-agent** is an orchestration framework built around MCP as the primary tool interface, handling server lifecycle, multiserver routing, and prompt context—useful when connecting to several MCP servers and integration code becomes the bottleneck.

---

## Layer 4: Browsers and Computer Use

Two architectural approaches exist: DOM-driven and vision-driven.

**Browser Use** is the Python default with 50,000+ GitHub stars, giving the LLM full browser control through an agent loop. Every step costs an LLM call, so production teams "cache the repeated 80% in Playwright" and reserve Browser Use for the 20% needing reasoning.

**Stagehand** is the TypeScript, MIT-licensed SDK from Browserbase, built on Playwright. Four primitives let developers use AI only for steps needing reasoning. Version 3 (February 2026) rewrote the engine on Chrome DevTools Protocol and ships 44% faster. Production deployment runs through Browserbase's managed cloud.

**Skyvern** is vision-first, using a three-phase pipeline (planner, actor, validator). It scores 85.85% on WebVoyager 2.0, strongest on form-filling where DOM is unreliable (canvas elements, React virtual DOMs in iframes, antibot machinery). The production pattern wires both approaches: DOM-driven as the primary path, with vision-driven as the escape hatch.

---

## Layer 5: Coding Agents and Sandboxes

Coding agents ship with three unique components: a sandboxed filesystem, terminal access, and a browser tool.

**OpenHands (formerly OpenDevin)** is the production-grade autonomous option with 72,000+ GitHub stars and an $18.8M Series A, used at AMD, Apple, Google, Amazon, Netflix, and NVIDIA. Its event-stream architecture moves through four states per loop. It scores 53%+ on SWE-bench Verified with Claude 4.5 and up to 72% with Claude 4. The agent has shell access, so review must happen at the PR level.

**Aider** is the terminal-native original with 35,000+ GitHub stars and 13,100+ commits. It's Git-integrated by design. Its Architect/Editor mode splits work between a stronger planning model and a cheaper coding one, cutting costs 30-40%. It scores 32% on SWE-bench Verified with Claude 4.5 but "ships fewer surprises because every action lands in Git." It's terminal-only with no IDE integration.

**Cline** is the VS Code-native option with 38,000+ GitHub stars. Plan Mode and Act Mode separate intent from execution, making every action reviewable before touching the codebase. It's IDE-locked, so JetBrains or Neovim teams should look elsewhere.

The article notes most teams running production coding agents in 2026 run two: one commercial (Claude Code, Codex) for hard tasks and one open source for flexibility and outages.

---

## Layer 6: Evals and Observability

Tracing captures every LLM call, tool invocation, and cost, indexed by user and session. Evals are reproducible test suites with pass/fail criteria. The article calls skipping this layer "the most expensive mistake in agent engineering."

**Langfuse** is the open source observability default with native integrations across LangGraph, CrewAI, OpenAI Agents SDK, and Mastra. It's open core with a generous self-hosted tier. Managed retention, SSO, and advanced eval features run on the SaaS plan.

**Arize Phoenix** is OpenTelemetry-native, so agent telemetry flows into existing Grafana, Datadog, or Honeycomb dashboards. It's strong on RAG evals but doesn't ship opinionated agent-specific defaults.

**Inspect AI** is the UK AI Security Institute's open source eval framework, originally for safety evals (jailbreak resistance, PII leakage) but now used for capability benchmarking. It's for offline evaluation only.

The engineering lesson: "Wire tracing in on Day 1, before the first user."

---

## Layer 7: Models and Inference

**vLLM** is the production serving default for open-weight models, using PagedAttention for KV cache management and continuous batching. It's GPU-only and optimization-heavy.

**Ollama** is the local default with one-line install and an OpenAI-compatible API. It's not a production serving layer past a single user.

**llama.cpp** is the engine Ollama runs on—pure C++ with no GPU dependency, running on CPU, Apple Silicon, Raspberry Pi. It defined the GGUF file format. CPU throughput is well below GPU serving, making it best for local and offline workloads.

**SGLang** is the newer challenger that caches computation for shared opening prompts and enforces JSON schema inside the inference engine. On agent workloads it benchmarks faster than vLLM, but has a smaller community and is less battle-tested.

---

## The Cheat Sheet

The article provides a table showing each layer's statefulness, lock-in risk, and demo-to-production migration time:

| Layer | State | Lock-in | Demo→Prod |
|-------|-------|---------|-----------|
| Orchestration | High (schemas, state) | Medium (node semantics) | Weeks |
| Memory | High (vectors, graphs) | Low (vendor API) | Weeks |
| Protocol/Tools | None (config) | Low (MCP is standard) | Hours |
| Browser/CUA | Low (scripts) | Medium (cloud dep.) | Days |
| Coding Agents | Low (sandbox) | Medium (tool config) | Days |
| Evals/Observability | Medium (traces) | Medium (ingestion) | Days |
| Inference | None (stateless) | Low (OpenAI API) | Hours |

The core reframe: "An agent's toolkit is seven small bets, each with a single dominant constraint, and each made independently."
