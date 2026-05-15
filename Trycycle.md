# Trycycle

Dan Shapiro's hill-climbing skill for Claude Code, Codex CLI, Kimi CLI, and OpenCode. Takes any request -- from "make the button blue" to a 20-page specification -- and runs it through plan-strengthen-review loops using fresh agents at every stage. The key insight: each review uses a new agent with no memory of previous rounds, so stale context never accumulates.

---

## Key Themes

#agentic-coding #guardrails #orchestration

The pipeline:

1. Write a plan
2. Send to a fresh planning issue finder (same task input and repo context)
3. If issues found, deepen with the same reviewer, then hand findings to a fresh planning synthesizer
4. Fresh reviewer checks the synthesis -- up to 5 review/synthesis rounds
5. Once the plan is locked: build test plan, build code
6. Fresh reviewer produces structured observation packet
7. Fix what the packet shows -- up to 8 review rounds
8. Plan reconsideration at round 4 and every 2 rounds after

Philosophy: take any request of any size, avoid asking the user questions, prioritize zero bugs even at high token cost. Use your best judgment for anything not specified.

The "fresh eyes" at each review stage is the central innovation. It's the anti-pattern to context accumulation -- where most agents get worse as conversation grows, Trycycle gets better because each reviewer starts clean. This is related to [[Prefix Effects]] but works against it: fresh agents can't be biased by the codebase's existing naming conventions because they see only the plan and the diff.

The pipeline itself is a DOT-file-style workflow in the spirit of [[The Dark Factory is a DOT File]]. Shapiro credits Jesse Vincent's Superpowers methodology (same lineage as [[Verbose Deployment]]) and StrongDM's dark factory work.

Works cost-effectively on Deepseek v4 via OpenCode.

## Critical Analysis

Trycycle's strength is its disciplined use of disposability. Most agent workflows try to preserve context because context is expensive; Trycycle inverts this by treating fresh perspective as more valuable than accumulated context. This is a genuine insight.

The weakness: up to 5 planning rounds and 8 review rounds means potentially 13+ agent invocations for a single task. The token cost is real (mitigated by using cheap models like Deepseek v4). And the "avoid asking the user questions" philosophy means Trycycle will make assumptions that may be wrong. It works best with detailed specs or vague requests -- the middle ground ("I care about the details but didn't specify them") is where it struggles.

---
*Sources: [[raw/trycycle]]*
*Last updated: 2026-05-14*