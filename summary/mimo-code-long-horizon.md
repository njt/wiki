---
url: https://mimo.xiaomi.com/blog/mimo-code-long-horizon
title: "MiMo Code: Scaling Coding Agents to Long-Horizon Tasks"
author: Xiaomi MiMo Team
date_fetched: 2026-06-11
date_published: 2026-06-10
topics:
  - agent-architecture
---

# MiMo Code: Scaling Coding Agents to Long-Horizon Tasks

MiMo Code is a terminal-based coding agent designed for extended automated programming tasks, tackling the challenge of "how to maintain decision quality and state continuity over dozens or even hundreds of execution steps." The article organizes its technical design around three pillars: **computation, memory, and evolution**.

Product links: Try it now, Docs, GitHub (MIT-licensed, built on OpenCode)

---

## 1. Design Motivation

The authors observe that coding agents typically place a language model inside a runtime calling it in a loop — the model reasons and decides, while the runtime manages tools, state, and input assembly. The model is stateless; "all continuity is provided by the runtime."

Two core problems emerge as task turns increase:

1. **Context window exhaustion** — tool outputs, code, and error logs eventually fill any window. Simple compression "continually reinforces nearby information while weakening distant information," creating a dilemma like recurrent models (Mamba): it has state but "cannot look back on demand." The solution is not better compression but "an explicit storage-and-retrieval mechanism."

2. **Declining instruction-following** — even with large windows, "useful constraints and intentions are diluted by large volumes of tool output."

The team identifies bottlenecks across three time scales: single-turn quality is constrained by **computation**; multi-turn continuity within a session by **state management**; cross-session improvement by the **mechanism for distilling experience**.

---

## 2. Computation: Scaling Single-Turn Reasoning

The article frames reliability as a compounding problem: error rates accumulate over dozens/hundreds of steps, and external corrective signals are often absent during long-horizon execution.

### 2.1 Parallel Sampling and Selection (Max Mode)

At each turn, N candidate solutions (default N=5) are generated in parallel. Each completes reasoning and tool-call planning but does *not* execute. The same model judges all candidates, selecting the best for actual execution. Temperature is set to 1 by default, so "five independent samples almost never produce identical results." On SWE-Bench Pro, Max Mode "improves performance by 10–20% compared with single sampling, at the cost of roughly 4–5 times the token consumption." This is experimental and must be manually enabled.

### 2.2 Independent Completion Verification (Goal)

"Max Mode addresses 'doing it right'; Goal addresses 'finishing it.'" The user defines a natural-language stopping condition (e.g., "all tests pass and the code has been committed"). When the agent tries to terminate, "an independent model call" reviews the full conversation history to check whether the condition is met. If not, it feeds back the gap. The verifier is independent, so "it does not develop an alignment bias toward the parts the agent has already completed."

False blocking (the condition is met but the verifier thinks it isn't) is more common than false passing, often due to environment-related test failures. "Overall, the probability of an infinite loop is below 0.5%."

Max Mode (parallel, N× compute per step) and Goal (serial, more self-checking) are orthogonal and can be simultaneously enabled.

### 2.3 Tool-Call Syntax

The team notes that "some models (especially the GPT-5.5 series) have a relatively high formatting error rate" with structured JSON, XML performs slightly better, but a constrained command-line syntax uses fewer tokens and produces fewer formatting errors because models are trained on shell data. The syntax intentionally excludes pipes, redirection, and variable expansion. The approach "is to borrow the conciseness of the shell, not to give the model an uncontrolled execution environment." This format has not yet been migrated into MiMo Code.

### 2.4 Large-Scale Parallel Orchestration (Dynamic Workflow)

For massive tasks like migrating an entire project between programming languages, round-by-round tool calls become insufficient. The traditional approach (writing workflow into SKILL.md as natural language instructions) "systematically fails in complex workflows": context compression may swallow steps, models may skip stages, branching depends on model judgment rather than code guarantees, and "the same workflow may follow different execution paths across two runs." The root problem: "orchestration logic exists in natural language, and natural language is ambiguous, forgettable, and unverifiable."

Dynamic Workflow "turns orchestration logic from prompt into code." The main Agent generates a JavaScript script executed deterministically in an isolated sandbox. Sub-Agents are dispatched via `agent()`, concurrency via `parallel()`/`pipeline()`. "An `if` statement will not forget a branch, a `for` loop will not exit prematurely, and a barrier will not miss a sub-Agent." The model's judgment is reserved for code understanding/generation rather than flow control.

The implementation is compatible with Anthropic Dynamic Workflow core semantics, with extensions: `workflow()` allows scripts to call other scripts (reusable building blocks), each `agent()` result is synchronously written to disk for recovery after interruption, and files can be read/written inside the sandbox.

---

## 3. Memory: Maintaining State Continuity Across Multi-Turn Tasks

This section addresses the core challenge: "the context will eventually run out." The goal is to let "a logical session extend indefinitely while keeping each physical window bounded."

### 3.1 Cycle: The Basic Unit of an Unbounded Session

The runtime intervenes at fixed positions called **checkpoints** before the limit is reached. At each checkpoint, an independent writer subagent reads the conversation and writes a structured state file to disk, concurrently with the main agent. When the window nears the limit, a **rebuild** occurs: the old window is cut off, a new one opens, and persisted files are used as seeds to reconstruct context. The model perceives the conversation as uninterrupted. Each checkpointed sequence ending in a rebuild is a **cycle**; "that chain has no maximum length."

### 3.2 Why Extract Early

The team argues that delaying extraction until the window is nearly full is "exactly backwards." First, model capability degrades under high context utilization ("lost in the middle") — "asking the model to perform the most critical compression at the very moment when its compression ability is degrading is a bad trade-off." Second, extraction requires space within the window. Checkpoints are triggered at approximately 20%, 45%, and 70% of the configured budget, each an incremental update. The final rebuild near the limit is not a rushed compression but the moment when accumulated structured records become working context.

### 3.3 Writer: An Extractor Independent of the Main Agent

The team found that asking a debugging model to also maintain structured logs causes it to do both tasks worse. The design constraint: **the main agent does not maintain its own memory.** Extraction is entirely moved out of the main loop, triggered by the runtime, performed by an independent writer subagent that doesn't share the main agent's attention or token budget.

The writer produces a checkpoint file with 11 fixed fields (current intent, next action, working constraints, task tree, current work, involved files, cross-task discoveries, errors and fixes, runtime state, design decisions, miscellaneous notes) and updates project-level memory as needed. Single-writer invariant prevents inconsistent state from concurrent writes.

### 3.4 Four Layers of Memory

- **Session memory** (checkpoint.md): Lives only within the current logical session.
- **Project memory** (MEMORY.md): Persistent project-level knowledge. When an observation stabilizes across multiple checkpoints, the writer promotes it from session to this layer.
- **Global memory**: User-level preferences across projects.
- **History**: Full SQLite trace of every message and tool call. "When a detail cannot be found in structured memory, the agent uses the history tool to trace back to the original record."

The relationship: upper layers are more refined, persistent, and smaller; lower layers are more complete, larger, and slower. The writer distills upward; history serves as fallback below.

The main agent has read-only access to structured files, with one exception: **notes.md**, a session-level free-form scratchpad. The main agent can append scattered findings; at each checkpoint the writer reads it, routes content into appropriate structured fields, and clears it.

### 3.5 Rebuild Injection

During rebuild, persisted files are assembled into a layered prompt. The approximate order: task list → session checkpoint → verbatim slices of recent user messages (to prevent "the writer's rewriting from drifting away from the user's original intent") → project memory → global memory → notes → an index of memory file paths → a tail reminder. "Even if every section reaches its limit, the total injected content is kept within roughly 65K tokens." The agent continues working directly without re-reading already-processed files.

---

## 4. Evolution: Continuous Improvement from Experience

This addresses cross-session learning: "if all experience is lost after each session ends, the agent can never accumulate from past work."

### 4.1 Project Memory

A project-level Markdown file stores knowledge across sessions: project background, user-specified rules, architectural decisions with rationale, and repeatedly verified technical facts. Files were chosen over pure vector databases for **reviewability** — users can see what's been remembered, delete incorrect entries, and modify outdated knowledge using standard read/write tools. The background writer has code-enforced path restrictions; out-of-bounds writes are rejected.

### 4.2 Memory Maintenance (Dream and Distill)

**Dream** triggers every 7 days. An independent agent reads historical sessions and the existing memory file, performing "merging, deduplication, path-validity verification, and compression" — converging scattered memories into a compact current-state representation.

**Distill** triggers every 30 days. Also performed by an independent agent reading historical sessions, but its focus is process, not knowledge. It identifies recurring work patterns and solidifies them into "reusable skills, CLI commands, custom agents, SOP documents, and similar artifacts."

---

## 5. Evaluation

### 5.1 Offline Benchmarks

MiMo Code + MiMo-V2.5-Pro outperformed Claude Code + Claude Sonnet 4.6 across three benchmarks (SWE-Bench Pro, etc.). However, the article notes these benchmarks "still measure one-shot problem-solving ability on individual repository-level issues," while MiMo Code's core design advantages (multi-turn memory, background state maintenance, completion verification, cross-session evolution) "mainly show their value in real development scenarios that continue for dozens of turns."

### 5.2 Human Double-Blind A/B Testing

An internal beta covered "576 developers and 474 real private repositories, producing 1,213 A/B pairs with clear win/loss judgments." MiMo Code and Claude Code were compared under the same target model. When execution steps were within 200, win rates were close to 50%. "When the number of steps exceeds 200 (including multi-turn user interaction), MiMo Code's win rate rises to over 65%."

---

## 6. Usage

Installation via one-line curl command or npm:

```
curl -fsSL https://mimo.xiaomi.com/install | bash
npm install -g @mimo-ai/cli
```

First launch guides users through model access selection: MiMo Auto (free for one month, based on MiMo-V2.5 with 1M-token context), Xiaomi MiMo platform login, import from Claude Code configuration, or custom model.

---

## Key Claims & Insights Summary

- The core thesis is that scaling coding agents requires addressing three distinct time scales: computation (single-turn), memory (multi-turn sessions), and evolution (cross-session).
- The main agent should not manage its own memory — extraction must be delegated to an independent subagent.
- Early checkpointing (20%/45%/70%) outperforms waiting until windows are nearly full.
- Orchestration logic should be code (JavaScript) rather than natural-language prompts to guarantee deterministic execution.
- The human A/B test shows MiMo Code's advantage emerges primarily beyond 200 execution steps (65%+ win rate), validating the long-horizon focus.
- All memory layers are file-based for user reviewability rather than relying on opaque vector databases.
