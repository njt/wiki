# The AI-Native Large-Scale Agile Manifesto

An arXiv manifesto paper that aims the AI-native argument at the organizational layer rather than the individual developer: large-scale agile fails because meetings, handoffs and static documents are the coordination mechanism, so replace them with shared living knowledge, parallel AI execution, verification-first assurance, orchestrated agent workforces, and reusable-but-adaptable blueprints.

---

It reads the obvious failure modes of SAFe and LeSS — coordination overhead, knowledge silos, phase gates — and asks what happens when execution cost collapses: "You no longer need to carefully gate every step to protect the cost of human effort." The six principles form a coherent system, and the strongest one is the governance inversion: "Human in the Loop, where every step requires human approval, becomes Human in Control. The difference is subtle but critical: you retain full authority over the outcome without becoming a bottleneck in the process." That is the cleanest formulation yet of why per-step approval is the wrong oversight model when correction is cheap.

## Key quotes

> Humans set the intents, and a context-aware, knowledge-rich coordination approach handles interactions with an AI-agentic workforce, eliminating unnecessary meetings.

The manifesto's wager is that coordination should live in artifacts, not calendars — the same bet the wiki's [[Intent Is the Interface]] and [[Organizations Need Decision-Grade Knowledge]] notes make at smaller scale.

> The traditional developer writes code. The AI-native developer defines intent. ... Engineers become designers of environments, guardrails, and controls. This is not a reduction in skill; it is a different and arguably harder skill.

> Human in the Loop ... becomes Human in Control. Corrections after the fact are cheap. Waiting for approval at every step is expensive.

> Today, most of that knowledge lives in static documents ... These artifacts have a half-life measured in weeks. The moment they are written, they begin to decay.

The fix is AI-ergonomic living specs that agents both consume and maintain — "The artifacts stay alive because the system that uses them also maintains them." This is the strongest statement in the wiki of the thesis behind [[Your .md File Is Not a Specification]] and [[Spec-Driven Development]]: the spec is the durable artifact precisely because it is the thing both humans and agents read.

> Tests are generated from specifications before any code is written. ... the tests become a contract that the code must satisfy.

Verification-first inverts the testing order and outsources BDD's historical scaling problem (scenario explosion across teams) to agents that detect overlap and contradiction.

> AI agents organized not as a single all-knowing assistant but as a coordinated team of specialists. ... By splitting work across scoped subagents, with session resets, delegation protocols, and externalized context, the system stays sharp regardless of how large or complex the project becomes.

The Persona / Agent / Skill triad and the bounded-context argument against context overflow echo [[Multi-Agent AI Systems Are Organizations]] and the supervisor patterns in [[Agent Orchestration]].

## Themes

#concept #pattern #specification #orchestration

## Opinionated take

This is a vision paper, not an empirical one — there is no case study, no measurement, no team that actually ran this at scale, and the references are thin on the organizational side. As a manifesto it is honest about that ("we are uncovering better ways ... by doing it"), but its weakest joint is the semantic layer: four fused knowledge graphs kept continuously up to date by the agents that use them is a research project wearing an org-chart costume, and anyone who has maintained an enterprise ontology will read §3.1 with a raised eyebrow. The blueprint section's role list (Security Expert, Verification Expert, UX Expert as personas) also quietly reintroduces the specialization-and-handoff structure the manifesto spends §2.1 dismantling — the handoffs just happen between agents instead of humans, which may or may not be cheaper.

Still, the paper earns its place for two things: the Human in Control / Human in the Loop distinction, which is the right governance vocabulary, and the framing of large-scale agility as an *economics* problem — when iteration is nearly free, phase gates stop paying for themselves. That economic argument is the most defensible part, and it's the part practitioners actually report.

The "dark software factory" endgame it gestures at is treated more skeptically elsewhere in this wiki — see [[Don't Fear the Dark Factory]] and [[Software Factories, Light and Dark]], which both insist autonomy cannot exceed cheap, unfakeable verification. This manifesto's verification-first principle is exactly that rule, stated as dogma rather than as a hard-won lesson.

---
*Sources: [[raw/2605-07717v2]], [[summary/2605-07717v2]]*
*Last updated: 2026-10-18*
