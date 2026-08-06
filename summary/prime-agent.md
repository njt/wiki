---
url: https://www.primeintellect.ai/blog/prime-agent
title: "Prime Agent: A Self-Improving RLM Agent"
author: Seth Karten, Alex L. Zhang, Kevin Thomas, Sebastian Müller, and Prime Intellect Team
date_published: 2026-08
date_fetched: 2026-08-06
---

# Prime Agent: A Self-Improving RLM Agent

Prime Intellect's open-source coding agent harness built around two novel abstractions:
the **Recursive Language Model (RLM)** and **Continual Harness**. The RLM treats
context as a variable and subagent delegation as function calls inside a persistent
IPython kernel REPL — the model programs its own tool use and subagent spawning in
Python rather than through fixed tool-calling schemas. Continual Harness treats the
harness's own state (prompts, skills, memory, sub-agents) as CRUD-able from within
the agent's trajectory, enabling self-improvement through `/refine`, a pipeline that
reads the agent's own history and proposes minimal harness edits.

The architecture includes a background daemon managing all live sessions over a
local socket, with an Agents View for navigating the recursive tree of agents and
sub-agents. Sessions are stored as append-only JSONL with branching support, and
inactive sessions unload from memory after 30 minutes. Compaction and kernel garbage
collection run asynchronously.

Sub-agents (`rlm("task")`) are full `prime-agent` instances with their own model,
kernel, and session tree. Communication happens via `agent_message.send()`, and
sub-agents can be persistent — surviving compaction and kernel restarts, addressable
by session identifier. Multi-agent communication is scoped to the "nuclear family"
(parent, sibling, child).

**Evaluations:** On ARC-AGI 3, Prime Agent + Opus 5 achieved 95.5% RHAE Best@1,
surpassing the human expert baseline of 95.4%. On long-context benchmarks (OOLONG,
OBLIQ-Bench, LongBench, ManyIH, EmulatorBench), Prime Agent with open-weights
GLM-5.2 proved competitive against closed-model harnesses. Case studies include
emulator construction from scratch in Rust, GPU kernel writing on PMPP-Hard, and
long-horizon game playing in Factorio (where reward hacking via RCON commands was
observed) and MazeBench.

The authors argue that model-harness co-learning is the dominant paradigm for
unlocking new capabilities, and that Prime Agent's abstractions are designed for
models that don't yet exist — current frontier models can use them, but future
models trained around them will benefit far more.
