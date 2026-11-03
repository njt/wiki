---
url: https://knlb.dev/essays/surrender
title: "Working Notes: Cognitive Surrender"
author: Kunal (knlb.dev)
date_fetched: 2026-11-03
date_published: 2026-10 (October 2026 per text)
topics:
  - agent-coding-workflow
  - software-engineering-craft
---

Kunal's essay on *cognitive surrender*: uncritically accepting whatever coding agents output without actually thinking, exploring, or considering alternatives. He finds the mode fast, seductive, and corrosive — suboptimal decisions, technical debt, and eroding familiarity with his own systems.

## Why surrender is seductive

- Programmer laziness (decades of training toward the shortest path) finds handing the problem over irresistible: "If I can spin up a software factory, why wouldn't I?"
- Peer pressure and game theory: if colleagues appear to have surrendered, spending time *understanding* looks like falling behind; agent-generated PR volume looks like productivity.
- Models' expressed confidence (an RLHF artifact, he suspects) makes their answers easy to take at face value — and they're oddly blasé about admitted mistakes.
- OSS projects under time pressure accrue complexity to the point that agents feel like the *only* sane way to interact with the code — a self-reinforcing broken-windows loop.
- Complexity's feedback loop is attenuated: slop doesn't bite immediately, it compounds invisibly. Plus the hype cycle of Anthropic-loop talks.

## The cost

He describes a Gell-Mann-amnesia inversion: with expertise he can simplify agent output aggressively; without it, agents seem like the smartest engineers alive — until they get stuck making a directory and his assumptions reset. Maximal agent use (October 2026 SOTA) yields complexity explosions: mostly-working systems with surprising edge cases, shipped at scales that break the tooling itself. The deeper, persistent cost: *mechanical sympathy* never develops. His only debugging option becomes "spin up one more session and hope," and each automated fix erodes understanding further — a self-reinforcing loop. He leans on Bainbridge's *The Ironies of Automation* (1983): automation makes the failures harder precisely because you never practised the easy tasks.

The accountability framing: "I'm responsible and accountable for what I build, not the agents. Surrendering agency and not thinking basically makes me accountable without any meaningful ownership."

## The counter-practices

He accepts he can't have his cake and eat it: agents drive build time toward zero but comprehension time hasn't moved. His tactics:

- A personal **vibe limit**: the complexity of change he can make with sufficient confidence using agents; explicitly triage which projects are safe to surrender and which need his own time, recalibrating against outcomes with every model drop.
- Watch for the "repeatedly tapping yes, continue" warning sign — then either fully delegate (better sandbox, cheaper sanity checks) or re-engage.
- **Choose to drive** on projects he cares about: every change made by hand, agent as editor only. His notebook's AGENTS.md says: never edit directly, always wait for confirmation. Echoes his old Stack Overflow discipline: never copy-paste; retype while cross-checking docs.
- Agent-built comprehension scaffolding: 4-column PR views (diff, diff, agent commentary, his comments), "make the change easy, then make the easy change" reviewable stacks, debugger sessions until the real execution path is felt, CLI tools that explain their own flags.
- Christopher Alexander's decomposition argument: lightly coupled, cohesive subsystems adapt quickly — good design serves comprehension regardless of who writes the code.
