# Cord

A protocol for coordinating trees of AI agents with runtime task decomposition. Instead of developers hardcoding agent workflows (LangGraph's static graphs, CrewAI's fixed roles), Cord lets agents dynamically build task trees using five primitives: spawn (independent subtask), fork (dependent subtask with sibling context), ask (human input), complete, and read_tree. Built on Claude Code CLI + MCP + SQLite in ~500 lines of Python.

---

## Key Quotes

> "Real work isn't one task. It's a tree of tasks with dependencies, parallelism, and context that needs to flow between them."

> "Why are we still hardcoding the decomposition?" when current models excel at natural planning.

> "The protocol is the contribution. This repo is a proof of concept."

## Key Themes

#multi-agent #coordination #task-decomposition #protocol #mcp

The spawn vs. fork distinction is the key innovation. Spawn creates a clean-slate subtask (cheap, independent). Fork injects sibling results into the new task (expensive, but necessary when tasks depend on each other). This gives agents runtime control over how context flows through the task tree -- something no hardcoded workflow graph can do.

The behavioral testing results are encouraging: 15 tests showed agents naturally discovering read-verify cycles, escalation patterns, and error recovery through task respawning -- all without being explicitly programmed. Clear tool descriptions plus dependency semantics were sufficient for correct protocol use.

This connects to [[Experience Design for Agents]]'s emphasis on context management and [[Elements of Agentic Systems Design]]'s Coordination and Reasoning elements. It directly challenges [[What I learned building an opinionated and minimal coding agent]]'s argument against sub-agents -- Cord's counter is that sub-agents with a clean protocol and dependency tracking are different from black-box orchestration.

## Critical Analysis

Strong: the five-primitive protocol is elegant and implementation-agnostic (could run on any LLM, any backend, even with human workers). The behavioral testing showing emergent patterns validates the protocol design. ~500 lines of Python is refreshingly small for a coordination framework.

Weak: the proof of concept runs on Claude Code CLI, which means you're paying for a full Claude Code session per subtask. At scale, this gets expensive fast. The dependency resolution is SQLite-backed, which limits distributed deployment. And the fundamental bet -- that models can reliably discover appropriate coordination structures -- may not hold for deeply unfamiliar problem domains where the task decomposition itself requires expertise.

The "protocol is the contribution" framing is honest and correct. Whether this specific implementation scales matters less than whether the spawn/fork/ask/complete/read_tree pattern becomes a standard for multi-agent coordination.

---
*Sources: [[raw/cord]]*
*Last updated: 2026-05-14*
