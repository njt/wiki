---
title: "Three Tier Memory"
url: https://arxiv.org/abs/2602.20478
date_fetched: 2026-05-14
section: "AI-enhanced Coding"
topics:
  - agent-memory-and-context
---

# Three Tier Memory - Aristidis Vasilopoulos

## Problem
"LLM-based agentic coding assistants lack persistent memory: they lose coherence across sessions, forget project conventions, and repeat known mistakes."

## Main Contribution
A three-component infrastructure developed during construction of a 108,000-line C# system:

1. **Hot-memory constitution** (660 lines, always loaded) - Encodes conventions, retrieval hooks, and orchestration protocols
2. **Specialized agents** - 19 domain-expert agents (9,300 lines total) invoked per task
3. **Cold-memory knowledge base** - 34 on-demand specification documents (~16,250 lines) queried via MCP retrieval server

## Research Scope
- **Development sessions analyzed:** 283
- **Case studies:** 4 observational studies showing how codified context prevents failures and maintains consistency
- **Scale:** Applied to a distributed system with significant complexity

## Key Innovation
The framework enables codified context to propagate across sessions, addressing scalability challenges for multi-agent projects that previous manifest file approaches couldn't handle.

## Publication Details
- Submitted: February 24, 2026
- License: Creative Commons BY 4.0
- Open-source companion repository with DOI
