# Real-Time Multiplayer Interfaces

Ramon Marc extends his "intent is the interface" thesis into the real-time domain: as AI agents become durable (holding state, running on heartbeats), human-agent interaction becomes a continuous two-way dialogue rather than one-shot prompting — and the design problem shifts from screens to shared protocols, with interruption, attention economics, and rehearsal as the new primitives.

---

## Key Quotes

> Cronjobs are having a comeback.

State turns tools into participants. Once both sides persist — agents on heartbeats, humans across sessions — every interface becomes implicitly multiplayer. The question isn't whether to design for real-time but how.

> For agents, reading and responding costs computation. And computation is buyable. For us it costs attention. And attention can only be outsourced, never purchased.

This is the asymmetric bargain at the heart of the design problem. Agents can scale compute infinitely; humans cannot scale attention. The interface isn't mediating information — it's mediating a fundamentally unbalanced resource. Prioritization and misunderstanding become the actual design surface.

> We stop designing fixed moments.

The core design shift: from pixels on a screen to the constraints, primitives, and protocols from which interaction emerges. The screen was a fixed moment we mistook for the product. The real product is the shared abstraction layer both parties negotiate through.

> If you think your attention is fractured, wait till you unleash the swarm.

Bidirectional interruption is the hardest human-element problem. Agents interrupting humans isn't a bug — it's the feature that makes the loop real. But the human cost of getting interruption wrong is catastrophic. This is the problem Marc treats as unsolved.

## Key Themes

- #concept **Real-time as dialogue, not speed** — Marc carefully distinguishes real-time from fast: it's about continuous mutual listening and the ability to interrupt, not low latency. A 30-second response can be real-time if the loop stays open; a 200ms response is one-shot if the channel closes after.
- #concept **Attention economics as design constraint** — The asymmetry between buyable compute and non-buyable attention is the fundamental constraint. Every design decision is a bet on where human attention should be spent, and "demand more attention" can never be the answer.
- #pattern **Derived interfaces** — Interfaces should be generated from intent and context, not designed for every surface. This extends Marc's earlier [[Intent Is the Interface]] argument into the temporal dimension: interfaces must also be generated from the interaction's rhythm, not just its intent.
- #pattern **Rehearsal as edge-case surfacing** — The fifth principle ("learns with us, does not calcify") is the wicked one. Adaptive interfaces can only respond to situations they've seen. Marc's solution: low-stakes, Duolingo-like rehearsal interactions that surface edge cases before high-risk moments, not during them. This is a genuinely novel approach to the cold-start problem in adaptive UX.
- #concept **Interruption as first-class design act** — In both directions. When should an agent interrupt a human? When should a human interrupt an agent? This is not an implementation detail — it *is* the interface. The protocol for interruption matters more than the content of any single interaction.

[[AI UX Patterns — User Transparency]] lands the same point from the product side: a visible "the AI is now driving" marker exists *so that* the human doesn't unintentionally interrupt an ongoing process. Nanz's framing (transparency as coordination, not just disclosure) is the missing precondition for interruption-as-design — you cannot interrupt correctly if you cannot see the agent's current mode.

## Critical Analysis

Marc is writing at the right altitude. Most "agent UX" writing either stays at the screen level (where should the chat box go?) or ascends to hand-wavy philosophy. This piece lands in the productive middle: design principles grounded in a specific architectural claim (durable state → multiplayer by default), with concrete open questions that admit they're unsolved.

**What's sharp:** The attention asymmetry is the cleanest framing of the human-agent interface problem I've seen. Everyone talks about "reducing cognitive load" but Marc identifies *why* load matters in a way that's specific to durable agents: because the agent can scale and you can't. The rehearsal concept is genuinely novel — it borrows from spaced repetition but applies it to interface reliability, and it's the only proposed solution I've seen for the adaptive-interface cold-start problem that doesn't boil down to "collect more data."

**What's missing:** The piece says "derived, not fixed" but doesn't wrestle with the privacy implications. You cannot derive context-appropriate interfaces without context, and you cannot have context without surveillance. Marc acknowledges this in his open questions ("capturing context without collapsing into surveillance") but sets it aside rather than engaging it. The tension between derived interfaces and privacy isn't a side question — it may be the central question, and the piece dodges it.

**What's meta:** The article was written by Claude "in an attempt to capture my voice." Given the subject matter — designing protocols for human-agent dialogue — the fact that the piece is itself an artifact of human-agent collaboration is either profound recursion or a category error. Marc's framing as an attempt rather than a claim suggests he knows this.

**What's worrying:** The comment from ✘. 𝑳𝒊 claiming to have "solved this" by abandoning semantic reasoning for "relation, pattern and structure" is either the most important footnote in the piece or pure crankery. The article doesn't engage it, which is the right call, but it's the kind of claim that either means nothing or means everything.

## Related Pages

- [[Intent Is the Interface]] — Marc's earlier piece on deriving interfaces from intent rather than designing for screens; this is the temporal/real-time extension of that argument
- [[Experience Design for Agents]] — Kemple's four-layer responsibility model; Marc's "appropriate load" maps to Kemple's "aligned autonomy levels"
- [[All Your Agents Are Going Async]] — Knill on the architectural gap when agents outlive connections; Marc is arguing the UX side of the same shift
- [[Agent Identity]] — Wolf & Reed on agents needing a stake, not just a log; maps to Marc's "mutual readability" — identity is what makes the loop two-way
- [[Agent-Native Architectures (Every)]] — Every's five design principles; Marc's "derived, not fixed" overlaps with their "emergent capability" principle
- [[Agent Memory and Context]] — The hub page for context engineering; Marc's "situational awareness" problem is the UX face of context management

---
*Sources: [[summary/real-time-multiplayer-interfaces]]*
*Last updated: 2026-07-05*
