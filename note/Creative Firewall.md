# Creative Firewall

Siddhi Sundar's framework for distinguishing prompts that carry genuine human authenticity from prompts engineered purely for output. The Creative Firewall is the boundary between imitation and intention; Trojan Prompts are the deeply personal prompts that prove the boundary exists.

---

## The Framework

Sundar's central claim: as AI tools get better at imitating creative styles, what distinguishes you isn't your technique — it's your lived experience. The Creative Firewall protects this. Trojan Prompts exploit it.

A Trojan Prompt is a prompt so anchored in personal memory, sensory detail, emotion, and specificity that an AI could not have written it independently. Sundar names five characteristics:

1. **Anchored in the Senses, Beyond Sight** — taste, smell, touch, sound
2. **Infuses Raw Emotion or Internal State** — "the ache of absence in every lamppost"
3. **Draws from Personal Memory or Specificity** — grandmother's worn pans, a specific kitchen window
4. **Embraces Juxtaposition or Tension** — "a neon butterfly landing on a forgotten gravestone"
5. **Implies the 'Why' Without Explaining It** — the rain that became theater, the weight of an empty notebook

She contrasts three prompt types in a matrix: Trojan Prompts (from memory/voice), AI Imitations (optimized but derivative), and Human-Guided Expansions (personal signals + technical scaffolding). Only the first "protects your voice at the root."

---

## Key Quotes

> "You're not designing a product, you're transmitting a memory."

This is the core reframe. Sundar isn't giving prompt-engineering advice — she's arguing that the most effective prompts come from a different place entirely: not craft, but autobiography.

> "Your prompt is still the torch. The firewall is the ember that only you can touch."

Even as agents become more autonomous, Sundar argues the initiating spark remains irreducibly human. The machine can fan the flame but can't strike it.

> "There is something in you that no machine can start."

The essay's closing line. Earned, not asserted — the preceding examples (the Temple of Poseidon, the grandmother's kitchen, the Chennai power outage) demonstrate rather than argue the point.

---

## Key Themes

#concept #tool #pattern #creativity

---

## Critical Analysis

**What's valuable here.** Sundar has identified something real that most AI discourse misses: the difference between prompts that work and prompts that *matter*. The Trojan Prompt concept is genuinely useful as a diagnostic — it gives you a way to check whether you're prompting from your own experience or from what you think the model wants. The five characteristics are concrete enough to apply.

**What's uncomfortable.** The essay frames this as almost spiritual — "the ember," "transmitting a memory." That register works for the creative audience she's writing for, but it borders on mysticism. The actual mechanism is more mundane: personal specificity forces the model out of its high-probability pathways into lower-probability regions of latent space. You get novelty not because the machine respects your authenticity, but because you've given it coordinates it hasn't visited before. Sundar gestures at this in the "What the Machine Does" section but doesn't name it.

**What's missing.** No discussion of whether Trojan Prompts *compound* — if you write ten of them, does the model converge on a voice that's recognizably yours, or does each one produce disconnected novelty? No exploration of whether this approach works for non-creative tasks (could a Trojan technical specification produce better code than a standard spec?). And the TM symbols on "Creative Firewall" and "Trojan Prompt" are branding doing the work of argument.

**The tension with compound engineering.** This wiki is full of evidence that [[Guardrails and Feedback Loops|deterministic enforcement beats prompting]]. Sundar's framework operates at the creative frontier where there is no ground truth to enforce — you can't lint a poem. That makes this a useful complement rather than a contradiction: Trojan Prompts for the generative spark, mechanical guardrails for everything that follows.

---

## Cross-References

- [[Code Field]] — Another counterintuitive prompting approach: inhibition over instruction. Sundar's "implies the why without explaining it" overlaps with Code Field's negation-based approach
- [[Talking to Transformers]] — Domain language as compression. Trojan Prompts are extreme domain language where the domain is your own life
- [[A Non-Anthropomorphized View of LLMs]] — Halvar Flake's counterpoint: LLMs are functions through ℝⁿ, not proto-minds. Useful grounding when Sundar's language gets mystical
- [[The Mundanity of Excellence]] — Excellence as qualitatively different choices, not quantitatively more effort. Trojan Prompts are qualitatively different, not just better-engineered
- [[Computer Use is 45x More Expensive Than Structured APIs]] — Vision agents need hyper-specific prompts to avoid silent failure. The specificity Sundar advocates has a structural, not just creative, justification

---

*Sources: [[summary/inside-the-creative-firewall]]*
*Last updated: 2026-05-14*
