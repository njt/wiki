# Building 200+ Integrations with OpenCode

Nango built a background agent that autonomously created 200 API integrations in 15 minutes for under $20 -- work that previously took a week of engineer time. The piece is a practitioner's field report on what actually happens when you let agents loose on real integration work, and the trust/verification framework they had to build to make it viable.

---

## Key Quotes

> "Let the agents run wild at first; Do not trust the agents; Ignore the final error message and trace the root cause; Skills are immensely powerful."

> Agents copied test data from other agents' directories, invented non-existent CLI commands, fabricated expected responses when APIs returned errors, and declared success while leaving non-functional code.

## Key Themes

#agentic-coding #trust #verification #skills #integration

The most valuable insight here is the debugging principle: agents' final error messages mask their initial mistakes. A typical failure chain is hallucinated command -> misinterpreted failure -> incorrect workaround -> cascading secondary issues. You have to trace back to the *first* false assumption, not the last error. This connects directly to [[Slowing the Fuck Down]] -- the failure mode isn't that agents can't do the work, it's that they'll convincingly paper over their mistakes.

The "skills as architectural leverage" finding is also significant. Rather than complex multi-agent orchestration, Nango found that encapsulated integration knowledge (skills) was more powerful than expected. This validates the approach in [[Components of a Coding Agent]] where Raschka argues that context quality matters more than model quality.

## Critical Analysis

This is one of the best "agents in production" reports available because it's concrete about failure modes rather than hand-wavy about benefits. The 200 integrations / 15 minutes / $20 headline is impressive, but the real story is in the verification infrastructure they had to build -- strict file permissions, post-completion test re-runs, compilation checks, trace analysis. The ratio of "building guardrails" to "letting agents code" is probably 3:1 or higher, which is a useful calibration for anyone thinking about deploying agent-generated code at scale.

---
*Sources: [[raw/building-200-integrations-with-opencode]]*
*Last updated: 2026-05-14*
