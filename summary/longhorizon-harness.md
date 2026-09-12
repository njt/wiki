---
url: https://github.com/AMAP-ML/LongHorizon-Harness
title: LongHorizon-Harness
author: Ziyu Ma, Hailang Huang, Shun Zou, Yong Wang, Shidong Yang, Yiming Hu, Fei Wei, XiangXiang Chu (AMAP-ML)
date_fetched: 2026-08-21
date_published: 2026-08
topics:
  - agent-architecture
  - guardrails-and-feedback-loops
---

# LongHorizon-Harness

by AMAP-ML, August 2026 (arXiv:2608.01964)

---

LongHorizon-Harness is an open-source (MIT) Python harness that wraps existing coding-agent CLIs — Claude Code, Codex, OpenCode, and DeepSeek Harness — in a durable **Manage → Execute → Audit** loop, so a single goal keeps working across desktop GUI apps and the terminal for hours at a time. It does not train a new model or replace an agent; it engineers the execution loop *around* one.

The loop is split across three roles. The **Manager** rebuilds each round from the original goal plus verified state, and emits the next bounded step (`gui`/`cli`/`done`/`blocked`/`ask`). The **Executor** starts with a fresh context and performs exactly that one step. The **Auditor** independently inspects the real files, UI, logs, and tests — never trusting the executor's own claim. Only results that pass independent verification become trusted task state; a rejected result is kept as evidence, not progress.

The design treats context as the harness's job, not the model's. Each role runs as a fresh episode (the harness even disables Claude Code's own memory and history), and cross-round context flows only through the manager's maintained task state/contract and the auditor reports it references — never raw trajectories. Auditors are enforced read-only by snapshotting the workspace before and after and invalidating the report if the auditor wrote anything.

Reported gains (same Qwen 3.7-Plus backbone, Claude Code execution backend): WeaveBench pass rate 51.8 → 80.7, OSWorld 2.0 binary completion 2.8 → 8.3 (3×), Terminal-Bench 2.1 69.7 → 77.2 with 24% fewer tokens. A React/FastAPI workbench gives per-role model selection, approval gates, mid-run instruction injection, and resume; `eval/` ships frozen reproduction suites for all three benchmarks.

*Source: [[raw/longhorizon-harness]]*
