---
url: https://github.com/outerloop-science/outerloop
title: "Outerloop — autoresearch agents that improve your benchmark"
author: outerloop-science / Agentic Learning AI Lab, NYU
date_fetched: 2026-11-02
date_published: 2026-08-05
topics:
  - coding-agents-and-frameworks
  - guardrails-and-feedback-loops
---

Outerloop is an open-source (Apache-2.0, Python 3.12+, ~22k lines in the kernel alone) background-agent system from NYU's Agentic Learning AI Lab that runs the research loop on your own repos: an author agent picks a hypothesis and writes code, the kernel runs the benchmark on your Slurm cluster or single GPU machine, re-measures the candidate against the base tree at the same seed, judge agents review the change and its claim before any pull request exists, and every attempt — including failures — ends in a written report. Nothing reports back to the vendor; you bring your own keys, compute, and GitHub App.

The system is contract-bound. A target repo opts in with a `.outerloop.yaml` declaring benchmarks (a command that prints a metric), budgets (GPU-hours, runs per week), allowed write paths, and a roadmap the agents read but never write. The loader hard-codes invariants no YAML can override: the contract file, the roadmap, and `.github/` are always forbidden, and outerloop can never target itself.

Architecturally it is a set of roles (author, judges, verifier, planner, steward) as declarative RoleSpecs over a single `Harness` seam (Claude Code, Codex, hermes-agent today), driven by a Slurm resubmission chain ("the tick") that leases state via compare-and-swap commits to a `state` git branch. Sessions get a scrubbed environment (Seatbelt on macOS, no HOME inheritance, no bot credential), a one-tool syscall surface (`.outerloop/syscall.json`), and measurement is a pure function of committed trees — cacheable and resumable across process death. PRs default to human merge; the benchmark's own eval code must sit outside the agent's allowed paths so a doctored ruler can't survive into a measured candidate.
