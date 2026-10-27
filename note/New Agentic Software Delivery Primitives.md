# New Agentic Software Delivery Primitives

Microsoft's framing of the agentic SDLC: the familiar Plan–Build–Verify–Observe loop survives, but each stage now feeds and draws from a shared knowledge layer, and the human jobs to be done change dramatically. The essay argues teams should transform the SDLC stage by stage — not all at once — and that memory is only valuable when it serves an answerable question.

---

## What it argues

The "describe what you want, agents build it" framing treats the middle as a black box. Instead, decompose the delivery system by its durable primitives: for each stage, the inputs, the human and AI jobs, the output artifact, and the success criteria. The bottleneck moves as capabilities improve, which is exactly why all-at-once transformations stall — you can't see what's working.

The table of stage artifacts is the essay's core contribution: Plan produces goals and acceptance criteria; Build produces a PR *plus the context needed to understand its choices*; Verify produces "auditable evidence of system behavior"; Observe produces real outcomes that feed the next goals. Trust is established at each stage and carried forward.

## Key quotes

> Each stage now draws from (and contributes to) a shared foundation of knowledge.

The knowledge layer as "the new glue" is the boldest structural claim. Today that knowledge lives in individual developers' heads and fragments as teams grow; the essay wants provenance, versioning, and separation of observation from interpretation — remarkably close to what Stack Overflow argues in [[Organizations Need Decision-Grade Knowledge]], though this is a vendor roadmap where that is a product spec.

> Similar to a database index or a materialized view, the goal of memory is to give an agent quick access to data from the past to improve future performance.

The most useful passage. Memory framed as an engineering artifact with a falsifiable test — does the agent do the job better with it? — rather than a mystical "memory" or "dreaming." This is the same restraint [[Maybe Coding Agents Don't Need a Bigger Memory]] applies: continuity and purpose-built retrieval, not raw quantity.

> Humans and a refined sense of taste are necessary to identify and produce lasting value.

> Even a loop that runs repeatedly without human intervention would still contain these responsibilities... Autonomy does not remove that structure.

A strong point: a fully autonomous loop still needs goals, evidence, delivery, and learning — the structure is intrinsic, not a transitional artifact of immature agents.

## Themes

#concept — SDLC stages as durable primitives; knowledge layer as the new glue
#pattern — stage-by-stage transformation; memory-with-a-purpose / materialized-view framing
#tool — Copilot coding agents, Copilot Code Review with skills and MCP

## Analysis

This is vendor strategy writing with a real analytical spine. The stage-decomposition argument is sound and matches what practitioners report — the bottleneck has migrated from building to verifying (see [[Agentic Verification]], [[Running an AI-Native Engineering Org]]). The honest admission that "faster creation has limited value if validation can't establish trust in the result" is more candid than most product-page prose.

Two soft spots. First, the knowledge layer is asserted, not designed: the essay names what it should contain but the concrete examples (Copilot agents, skills, MCP) are product placement, and nothing shows how "separating observations from interpretations" survives contact with real repos. Second, the taste argument leans on an article of faith — that human judgment will *concentrate* in planning and validation rather than atrophy. It acknowledges agents "will help us achieve our goals, but it is up to us" without addressing what happens when the skills for judgment rust from disuse.

Still, the materialized-view framing of memory is genuinely worth stealing, and the artifact table is a usable checklist for any team standing up its own delivery system.

## Related pages

- [[Organizations Need Decision-Grade Knowledge]] — strengthens the knowledge-layer argument with a provenance-first spec this essay gestures at but doesn't build.
- [[Agentic Verification]] — nuance: this essay agrees verification is the new human-heavy stage but adds the artifact framing (auditable evidence per stage).
- [[Maybe Coding Agents Don't Need a Bigger Memory]] — independently converges on the same anti-hoarding view of memory: purpose-built, evaluation-tested access to the past.
- [[Running an AI-Native Engineering Org]] — complicate: a field report where the bottleneck shift this essay predicts has already happened, including the process ossification it doesn't mention.

---
*Sources: [[raw/new-agentic-software-delivery-primitives]], [[summary/new-agentic-software-delivery-primitives]]*
*Last updated: 2026-10-28*
