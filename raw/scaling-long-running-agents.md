---
title: "Scaling Long-Running Agents"
url: https://cursor.com/blog/scaling-agents
date_fetched: 2026-05-14
section: "LLMs"
---

# Scaling Long-Running Autonomous Coding

Wilson Lin's research documents Cursor's experiments running multiple AI agents autonomously for weeks on complex coding projects, achieving over 1 million lines of code generation.

## Key Arguments

**Single Agent Limitations**: Individual agents excel at focused tasks but struggle with complex, multi-month projects requiring sustained effort.

**Coordination Challenges**: The team initially attempted flat structures with self-coordinating agents using shared files and locking mechanisms, which failed due to:
- Agents holding locks indefinitely, creating bottlenecks
- Risk-averse behavior avoiding difficult tasks
- Inefficient work patterns without clear accountability

**Hierarchical Solution**: The breakthrough involved separating agent roles:
- **Planners** continuously map codebases and generate tasks recursively
- **Workers** focus entirely on assigned tasks without inter-worker coordination
- **Judge agents** evaluate completion and restart cycles

## Notable Quotes

"Agents would hold locks for too long, or forget to release them entirely...Twenty agents would slow down to the effective throughput of two or three."

"A surprising amount of the system's behavior comes down to how we prompt the agents."

"The best system is often simpler than you'd expect."

## Concrete Results

- Web browser built from scratch: 1M+ lines across 1,000 files in one week
- Solid-to-React migration: 266K additions/193K deletions over three weeks
- Video rendering optimization: 25x performance improvement
- Running projects: Java LSP (550K LoC), Windows 7 emulator (1.2M LoC), Excel implementation (1.6M LoC)

## Model Performance Findings

GPT-5.2 outperforms competing models for extended autonomous work through superior focus and instruction-following. Different models excel at specific roles rather than universal application.

## Main Conclusions

Hundreds of agents can collaborate on single codebases for weeks with minimal conflicts. The system demonstrates that scaling through parallelization is feasible, though optimization remains incomplete. Prompt engineering matters more than system architecture alone.
