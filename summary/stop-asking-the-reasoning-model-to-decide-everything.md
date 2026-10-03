---
url: https://www.jeremydaly.com/stop-asking-the-reasoning-model-to-decide-everything/
title: "Stop asking the reasoning model to decide everything"
author: Jeremy Daly
date_fetched: 2026-10-03
date_published: 2026-09 (exact date not stated in fetched text)
topics:
  - agent-architecture
  - agent-memory-and-context
---

Jeremy Daly divides an agent's work by Kahneman's frame: code for what you know, retrieval for what you've learned, System 1 for what requires judgment, System 2 for what requires reasoning. The agent still coordinates, but its reasoning model shouldn't make every decision. Cheap typed judgments — TypeSafe's Jev (Noul/Choice/Score, $0.042/M input tokens, 70–500 ms, free outputs) and the open-source Laya decision model — could justify far more checks in the harness than current LLM-as-judge costs allow.

He tests this concretely. A synthetic signup-endpoint release-readiness check: eight parallel questions over one shared state, policy code routing on the answers. Jev separates the cases cleanly (coverage probability 0.08 for the missing malformed-input test, high implementation score retained); Laya's demo checkpoint barely moves its answers and fails even the complete case — it's early, but open and fine-tunable. A second test assesses candidate memories for relevance, role, scope and supersession; Jev keeps the old incident as historical context while the superseding decision becomes current guidance. He is candid about the caveats: probabilities moved across repeated calls (0.80–0.92 pattern confidence, brushing a 0.8 review threshold), Jev applied supersession it was told about rather than inferred, and TypeSafe's headline 193x faster / 444x cheaper benchmark doesn't establish correctness on your codebase.

The architectural payoff: separate narrow assessments give the policy code better inputs than a general approval, so each `route()` branch handles a distinct kind of uncertainty and can be tested independently — and an engineering leader can distinguish a model problem from a policy problem. His closing advice is to prepare now: map one consequential decision, make memory evidence traceable, and spend a fixed budget replaying the workflow against a System One candidate, reinvesting savings in more checks.
