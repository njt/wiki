# Understand to Participate

Geoffrey Litt's AIE talk — as filtered through Simon Willison's blog — names the central tension of agent-assisted development: you must understand your codebase deeply enough to remain an active collaborator, not a passive observer signing off on changes you no longer comprehend. The alternative is cognitive debt that compounds silently until you're debugging a black box written by a black box.

---

## Key Quotes

> "Understand to participate."

The title is the thesis. Litt's framing isn't "understand to verify" or "understand to review" — it's **participate**. The loss isn't safety; it's agency. When you stop understanding, you stop being a collaborator and become a rubber stamp. This is the same diagnosis [[Cognitive Debt]] makes from the organizational side: code has become cheaper to produce than to perceive.

> "A rich set of concepts in your mind to think creatively and fluently."

Litt's second claim is about *fluency*, not just comprehension. You need enough mental models loaded to riff — to see alternative approaches, to catch wrong turns before they compound, to direct the agent rather than follow it. This is what [[The Joy and Power of Understanding|Igor Roztropiński]] calls "force before multiplier": LLMs multiply whatever force you bring, but zero times anything is still zero.

> Without that fluency, the ability to contribute is "meaningfully limited."

The quiet implication: teams that outsource understanding to agents aren't just taking on technical risk — they're *ceding the ability to steer*. The agent becomes the de facto architect not because it's better at architecture, but because nobody else has the map.

## Key Themes

#concept #cognitive-debt #coding-agents #participation #fluency

### Cognitive debt as participation loss

[[Cognitive Debt]] frames the problem as organizational risk — code you can't debug, systems you can't modify, juniors who never develop intuition. Litt reframes it as a *personal* loss: the moment you stop understanding is the moment you stop being a collaborator. Same problem, different stakes. One threatens the project; the other threatens your role in it.

### The fluency bar is rising

As agents handle larger, more sophisticated changes, the minimum understanding required to stay in the conversation also rises. This creates a ratchet: agents get more capable → they make bigger changes → you need deeper understanding to follow those changes → but the agents are making the changes faster than you can absorb them. The gap accelerates.

### Simon's meta-point

Willison's post is a short recommendation, not a deep analysis — but his choice to amplify this specific talk is significant. The person who has shipped more agent-built projects than almost anyone, who argues you can learn languages by scanning agent output, who says "tests are free now" — this person is pointing at a talk that says *understanding still matters*. It's a boundary condition on his own methodology. [[Simon Willison — Engineering Practices That Make Coding Agents Work|His Pragmatic Summit talk]] pushes hard on "don't read the code if you can prove it works." Litt's talk is the counterweight: prove it works, sure, but if you want to *participate*, you still need to understand.

## Critical Analysis

**What lands.** Litt's reframing of comprehension as *participation* rather than *safety* is the sharpest contribution. The industry conversation has been stuck on "is it safe to not read the code?" — a question that admits engineering answers (tests, verification, sandboxing). Litt's question — "can you still participate?" — doesn't have an engineering answer. It has a cognitive one. And cognitive limits don't scale with tooling.

**The missing synthesis.** Willison's own work offers the beginning of an answer: [[Compound Engineering]]'s "add a system" approach, [[Loop Engineering]]'s meta-skill of designing systems that prompt agents, conformance-driven development with test suites as specs. These are mechanisms for maintaining understanding *at a different altitude* — not line-by-line comprehension, but system-level fluency. Litt's talk doesn't engage with this, and the synthesis between "understand to participate" and "don't read the code, prove it works" is where the real methodology lives. Nobody has written that synthesis yet.

**The fluency ratchet is the long-term threat.** If agents consistently produce changes larger than humans can absorb in the available time, understanding becomes structurally impossible — not a failure of discipline, but a failure of physics. The only escapes are (a) agents that explain better than they code, (b) architectures that make comprehension cheaper than production, or (c) accepting that human participation becomes strategic rather than tactical. Option (c) is where the industry is drifting. Litt's talk is valuable because it names what's being lost in that drift.

**Why this matters now.** It is July 2026. Coding agents are crossing from "writes functions" to "architects systems." The comprehension gap Litt describes six months ago is wider now and accelerating. This talk won't age — it will become more relevant every quarter.

---
*Sources: [[raw/understand-to-participate]]*
*Original: [simonwillison.net](https://simonwillison.net/2026/Jul/2/understand-to-participate), July 2, 2026*
*Last updated: 2026-07-05*
