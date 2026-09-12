---
url: https://github.com/spacedock-dev/razorback
title: "Razorback"
author: CL Kao (clkao@datarecce.io)
date_fetched: 2026-07-18
date_published: 2025-06
topics:
  - agent-architecture
---

Razorback is a Python CLI (`rk`) for reproducible agentic benchmark research, built on Harbor 0.6.6. It turns benchmark runs into defensible numbers through a pipeline of freeze, run, score, and audit.

The core idea is **sealed experiment identity**: a cryptographic hash over canonical JSON that pins model version, sampling parameters, workflow content, prompts, skill versions, and agent kwargs. Two researchers running the same inputs get identical hashes, making runs auditable and reproducible. A content-addressed freeze store gives each trial cell its own git repo for checkpointing and concurrent-safe resume.

Razorback enforces **fail-closed correctness**. Every Pydantic model uses `extra="forbid"`; every agent adapter rejects unrecognised kwargs with explicit errors. A running budget gate uses atomic file writes with `flock` to halt overspend before compute is wasted. Scoring computes honest stratified pass@1 — per-query Wilson CIs are reported at the cell level, but stratum-level CIs are deliberately left null because the mean of per-query proportions is not a binomial proportion.

**Leak protection** operates in three layers: runtime command blocking (curl, pip install of canonical-data libs), post-hoc taint scanning of agent trajectories, and coverage verification of subagent dispatch traces. Benchmarks are plugin-driven — the DAB plugin supports 12 DataAgentBench datasets with docker-compose sidecar databases — so new benchmarks can be added without touching Razorback core.

The author, CL Kao, also created Spacedock (the multi-agent first-officer/ensign workflow system). Razorback bridges Spacedock's orchestration model with Harbor's benchmark execution: the `spacedock_solver` agent kind dispatches Spacedock workers inside reproducible, auditable benchmark runs. Where Akmon proves agent sessions weren't tampered with, Razorback proves experiments can be recreated.
