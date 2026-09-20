---
url: https://www.warp.dev/blog/agent-self-improving-software-factories
title: "Closing the loop with self-improving cloud software factories"
date_fetched: 2026-09-20
topics:
  - agent-orchestration
  - guardrails-and-feedback-loops
---

Warp's engineering post argues that deploying coding agents should be treated as a measurement problem, not a prompting problem: organizations need a closed-loop "software factory" in the cloud where every agent run is traced, scored, and fed back into the factory's own definition. A factory is an automation loop around the SDLC — agents that triage, spec, implement, verify, review, and monitor — and the piece lays out five properties a serious one should have: defined as code (a `factory.yaml` manifest), cloud-hosted, API-first execution, built-in metrics and evals, and multi-model/multi-agent by default.

The distinctive contribution is the self-improvement architecture. Scorers grade agent runs (correctness, cost, verbosity) on a schedule; self-improvement agents read the graded runs, distill patterns, and propose diffs against the version-controlled factory definition, which humans merge as PRs. Because self-improvement from historical runs can't do A/B testing, a separate benchmark system runs reference tasks across candidate configurations and feeds the winning setup back into the code.

The metrics proposed — PR throughput, cost per PR, automation percent, estimated savings over human work — sit between DORA metrics and the inner loop of building software. The framing is "meta-engineering": the engineering team's real product becomes the infrastructure that tunes the system. It is also, transparently, a product document for Warp Factories; the argument doubles as its purchase justification.
