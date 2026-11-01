---
url: https://outerloop.science/papers/outerloop-2026.pdf
title: "Outerloop: Open and Trustworthy Multi-Agent Autoresearch"
author: Mengye Ren (Agentic Learning AI Lab, NYU)
date_fetched: 2026-11-02
date_published: 2026-11-01
topics:
  - guardrails-and-feedback-loops
  - security-and-sandboxing
---

Outerloop is an open-source "kernel" for autoresearch: coding agents that autonomously improve a repository's benchmark score. The paper's central claim is that the agent should never be both author and evaluator of its own work, because self-evaluation produces three failures: selection bias (picking the luckiest of many noisy runs), tampering (writing to the evaluation code), and unacceptable gains (real improvements obtained the wrong way — overfitting, bundled edits, side effects).

The system is built like an operating system: a small trusted kernel holds measurement, edit-scope enforcement, budget metering and publication, while agent sessions run as untrusted sandboxed programs with no repository credentials, no experiment hardware, and no knowledge of the measurement seed. Authors run experiments, sleep while jobs execute, and submit claims; the kernel re-measures each submission back-to-back on a seed it draws itself, credits only gains above a contract-defined noise floor, and routes every credit through two read-only judge agents (one for gaming, one for attribution). Credited results become pull requests and a results ledger.

Evaluated on a nanoGPT training-speed benchmark at matched compute and model, the paper's headline result is stark: three published systems (AIDE, Karpathy's autoresearch, ShinkaEvolve) all over-claim — their audited gains are far smaller than their reported ones, tracking the "luck" of their chosen seed — while the kernel's credited gains match a three-seed fresh audit within noise. A control run is the cleanest evidence: when the authors' own agents self-report, they over-claim just like everyone else; when the kernel measures, the over-claim disappears. Beyond trust, the kernel doubles as a controlled instrument: author model choice dominates outcomes, one agent matches a team at equal compute, agents largely ignore inter-agent messaging, and much of the "progress" on the benchmark is recall of the known Muon optimizer rather than novel discovery.
