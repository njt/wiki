# workgraph

A persistent task coordination system for human-AI organizations that centers work rather than agents. "Agents can come and go. The graph remains." Task graph stored as plain JSONL on disk (git-friendly, human-readable), with complete execution history, evidence tracking, claims/handoffs, and human judgment points as first-class operations.

---

## Key Quotes

> "Agents can come and go. The graph remains."

> "The bottleneck is no longer only execution. It is knowing what was done, what failed, what evidence exists, where judgment entered."

## Key Themes

#task-management #coordination #agents #work-graph #rust

The core insight is that the *work* is the durable artifact, not the agent doing it. This is a direct response to the problem described in [[Cognitive Debt]] -- if organizational memory is eroding because AI removes the friction that creates understanding, then you need an external record of what was done, why, and what evidence supports it.

Written in Rust with a TUI interface. Supports multiple AI executors (Claude, Codex, OpenAI-compatible). The JSONL-on-disk approach is deliberately low-tech -- git-friendly, human-readable, no database dependency. Worktree isolation prevents parallel agents from stepping on each other's files.

Poietic PBC was formed and organized entirely through workgraph, including company incorporation and grant drafting. That's a meaningful proof of concept -- real legal and financial operations, not just coding tasks.

Workgraph is the foundation for [[speedrift-ecosystem]] (which adds cross-repo observation and drift detection on top) and is philosophically aligned with [[ralph-ban]] and [[weft]] (simpler kanban-based approaches to the same problem). Also see [[Managing Agents via Kanban Boards]] for Geoffrey Litt's take on the coordination surface.

## Critical Analysis

The "agents can come and go" philosophy is the right abstraction for a world where agent capabilities change quarterly and no single agent framework lasts. The JSONL format is both a strength (portable, debuggable) and a limitation (no built-in querying, no relational operations). At scale, you'd want something like SQLite underneath. But for the current era of 1-10 agents on a project, the simplicity wins. The main question is whether workgraph can survive the coordination needs of truly parallel multi-agent work (20+ agents) or whether something more structured would be needed.

---
*Sources: [[raw/workgraph]]*
*Last updated: 2026-05-14*
