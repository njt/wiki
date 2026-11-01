---
url: https://github.com/guregodevo/memdoor/tree/main/examples/workflows/team-solve
title: "team-solve: five small models solve a programming problem together"
author: guregodevo (Memdoor)
date_fetched: 2026-11-02
date_published: 2026 (undated)
topics:
  - agent-orchestration
  - coding-agents-and-frameworks
---

A README for Memdoor's `team-solve` example workflow: five small-model agents (a planner, two ideators, a solver, a manager) plus a repair worker, each starting from a fresh context and communicating only through files on disk. A script running the sample tests decides success, not a model. The workflow is eight YAML files, no code, no model named in any step — whatever `/model` is pinned to answers, local models included.

The page reports a parity check against the GVS5H paper (Lee, 2026) on Qwen3.8-27B: re-running the authors' harness, this workflow scores 9.8/10 mean over five passes on ten LiveCodeBench Hard problems vs the authors' 9.6, and beats a single-call Fable 5 baseline (8.0). Honest caveats are given: ten problems is too small to separate 9.8 from 9.6, contamination can't be ruled out, and a median run took 40 minutes and $0.39 ($35.79 for 50 runs).

The topology is a DAG: plan → ideate-a/b in parallel → solve → test (command) → review → repair → verify (command). Two of the seven steps are plain commands; test output never fails the run, but verify's exit code does. Every agent prompt says "fresh context, read the files, do not rely on chat" — the paper credits that ledger discipline for the gain. The one lost problem is instructive: the manager invented a wrong counterexample and the team overwrote a correct program, a loop defect kept as-is from the authors' design.
