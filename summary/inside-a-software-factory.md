---
url: https://www.oreilly.com/radar/inside-a-software-factory/
title: "Inside a Software Factory"
author: Paul Iusztin (Decoding AI Magazine)
date_fetched: 2026-09-13
date_published: "2026 (not stated; written July 2026)"
topics:
  - agent-orchestration
  - agent-coding-workflow
---

Paul Iusztin's definitional essay on the **software factory**: the whole loop, the whole SDLC run with autonomy, built from eight stages in three buckets. *What to build*: triage/intake, brainstorming, planning (the stage he calls most important — the agent plans read-only against the codebase, AGENTS.md, and the context layer, and outputs tickets backed by documentation, an ADR log, and a glossary). *Building and checking*: an implement loop pairing a software-engineer agent with a QA agent, a three-step review, review-CI where failures trigger a fixing agent, and release. *Self-improving*: monitor/incident response feeding production signals back into triage, closing the loop. Orthogonal to all eight sits the **context layer**, which caps what brainstorming and planning can even consider — his argument for the LLM-wiki pattern (Karpathy's term; Factory's AutoWiki, LangChain's OpenWiki) over parsing the codebase raw.

The human-placement answer: you are indispensable at **brainstorming and planning**, agents own everything in between, and you return for the final PR review and merge. OpenAI's extreme case (~1M lines, ~1,500 merged PRs over five months, zero hand-written lines) is cited under "Humans steer. Agents execute." Economics follow: use the strongest model for planning because everything downstream depends on it — "Total cost is tokens × price, not model tier," and a weak plan makes cheaper models retry until they out-cost the expensive one.

The tempering is drawn from his own overbuild: Squid v1 chased full autonomy with one grand pipeline and collapsed the first time something went off-script — he couldn't debug, halt, or redirect it. The fix is two access modes: granular commands (grill the plan, implement one task, review one step) and an optional end-to-end chain, with planning kept permanently human-driven. He parallelizes only with local agents in worktrees, doubts the parallel-feature ceiling, and states plainly: "The bottleneck is me, and that's by design."

Build vs. buy maps to scale: everyone starts on a prebuilt harness (Claude Code, Codex, or open source OpenCode/Pi), small teams encode their process as skills and agents (his Squid, Matt Pocock's skills repository, the BMad method), and "you cross the buy line the moment engineers you don't personally supervise run agents" — when observability, tracing, and cost tracking become someone's full-time job, buy Factory.ai or Warp's Oz. The smallest builds, the middle buys, and the largest builds again. The essay opens by sneering at the "____ engineering" label arms race (loop engineering, graph engineering read "more like marketing talk than anything that solves real problems") and closes with the honest caveats: factories are far from fully autonomous, anyone claiming to have cracked it hasn't tested it or is selling it, and his own bottleneck "stubbornly stays at planning."
