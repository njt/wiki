---
title: "How Hightouch Built Their Long-Running Agent Harness"
url: https://www.amplifypartners.com/blog-posts/how-hightouch-built-their-long-running-agent-harness
date_fetched: 2026-05-14
section: "LLMs"
---

# How Hightouch Built Their Long-Running Agent Harness

## Article Overview
Examines Hightouch's engineering approach to building production AI agents for marketing automation, focusing on architectural solutions for managing context, planning, and execution in open-ended tasks.

## Key Arguments

**The Problem with Traditional Approaches**
Existing agent frameworks rely too heavily on Directed Acyclic Graphs (DAGs), which work for finite decision trees but fail for sparse, non-deterministic marketing tasks. Early naive implementations struggled because models would "give up" after reaching satisfactory answers and quickly exhausted context windows.

**Core Solution: Separated Planning and Execution**
Hightouch separates the model's planning phase from execution, allowing dynamic plan updates. The system includes tool calls like `make_plan`, `execute_step_in_plan`, and crucially, `update_plan`—enabling the agent to modify its approach when discovering unexpected data patterns.

**Context Management Through Delegation**
Two primary techniques address the persistent context bottleneck:

1. **File Buffering**: Large query results are written to temporary files with contextual pointers, allowing the agent to manage memory without losing information.

2. **Dynamic Subagents**: Complex subtasks spawn isolated LLM threads with dedicated objectives. These threads handle messy exploratory work independently, returning only concise summaries to the main agent—"like using scratch paper on a math test."

**Replacing Embeddings with Fanout Patterns**
Rather than vector databases, Hightouch dispatches hundreds of parallel API calls to smaller models (like Claude Haiku) for classification tasks. This brute-force approach proves more reliable and cheaper than traditional RAG systems for reasoning about multimodal data.

## Notable Quotes

"This agent needs to be a data scientist, a marketer, and a creative partner all in one."

"Most of them were too rigid, focusing more on the developer experience of chaining calls together than on solving the core problem of long-form, autonomous reasoning."

"The ugly truth is that creating a truly capable agent system today has less to do with scaling RL or exotic model architectures and more to do with the deeply unfussy work of actual context engineering."

## Main Conclusions

Practical agent engineering prioritizes context management over model architecture improvements. Success requires treating models as orchestrators that can manage their own reasoning processes through structured tool calls, strategically delegate work, and maintain awareness through context compression techniques. The work is fundamentally unglamorous but essential engineering rather than prompt optimization.
