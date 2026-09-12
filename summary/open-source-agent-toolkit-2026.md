---
url: https://www.oreilly.com/radar/the-open-source-agent-toolkit-in-2026/
title: "The Open Source Agent Toolkit in 2026"
author: Paolo Perrone
date_fetched: 2026-07-18
date_published: 2026-07-14
topics:
  - agent-architecture
---

A survey of the 2026 open source AI agent ecosystem, organized into seven
independent architectural layers. The core thesis: most production problems
have a solution, but each has been solved a dozen incompatible ways. The
dominant constraint for your system—latency budget, audit trail, model
portability, or language stack—determines which tool fits.

**Orchestration.** LangGraph is the Python production default (durable graph
execution, verbose setup). CrewAI optimizes for rapid prototyping (role-based
agents, no state schema) but lacks crash recovery. Pydantic AI is best for
single-loop agents returning validated data. Mastra is the TypeScript-native
option for Next.js apps. Vendor SDKs trade lock-in for zero-friction setup.

**Memory.** The context window is not memory. Mem0 provides hybrid
vector+graph memory with 92% lower latency vs. full-context approaches, but
lacks temporal reasoning. Zep/Graphiti tracks how entities and relationships
change over time, at higher cost. Letta (formerly MemGPT) treats memory like
an OS: RAM for active context, disk for archival, with the agent managing
promotion. Distinguish runtime state (mid-task scratchpad) from knowledge
memory (across-session learning).

**Protocols.** MCP is the de facto standard, supported by every major vendor
SDK and framework. FastMCP is the Python server framework; mcp-agent
orchestrates across multiple MCP servers.

**Browsers.** Browser Use (Python, LLM-driven, 50k+ stars) is best paired
with cached Playwright scripts for the repeated 80%. Stagehand (TypeScript,
MIT license) uses AI only for reasoning steps. Skyvern (vision-first,
85.85% on WebVoyager 2.0) handles form-filling where DOM is unreliable.
Production teams wire both DOM-driven and vision-driven paths.

**Coding agents.** OpenHands (72k+ stars, $18.8M Series A) is the autonomous
production option with event-stream architecture, scoring 53-72% on SWE-bench
Verified. Aider (35k+ stars) is Git-native, terminal-only, with an
Architect/Editor mode that cuts costs 30-40%. Cline (38k+ stars) is VS
Code-native with Plan/Act separation. Most teams run two coding agents: one
commercial, one open source.

**Evals and observability.** Langfuse is the open source default for tracing
LLM calls, tool invocations, and costs. Arize Phoenix is OpenTelemetry-native
for existing observability stacks. Inspect AI focuses on safety evals. The
article's engineering lesson: wire tracing on Day 1, before the first user.

**Inference.** vLLM is the production serving default for open-weight models.
Ollama is for local use only. llama.cpp runs on CPU and Apple Silicon without
GPU, defined the GGUF format. SGLang benchmarks faster on agent workloads but
has a smaller community.

Overall: the seven layers are independent decisions, not a vertical stack.
Teams shipping reliable agents pick the best tool per layer and accept that
integrating the seams is part of the job.
