# A Smart Bear Skills

Jason Cohen (founder of WP Engine and Smart Bear, author of *Hidden Multipliers*) packages the frameworks from his A Smart Bear blog as a set of Claude Code skills — "asb-skills" — that coach founders through positioning, pricing, and market-viability decisions. The differentiator is stated as a refusal: the skills don't answer questions, they interrogate you, pressing on vague claims until you produce your own best thinking. It's the most fully worked-out example of a distinct skill-design pattern — *interrogation over generation* — applied to business strategy rather than code.

---

## The Core Thesis: You Cannot Interrogate Yourself

The philosophical spine of the whole package is a claim about human cognition, not about AI:

> "A hard decision has one big problem: you cannot interrogate yourself. Denial and excuse-making are normal human behavior. This is why strategy documents are full of hope and supporting data, instead of the scary challenges the business really faces. You need an outside attacker to get past this."

The site's headline differentiator restates it as a product principle:

> "They don't think for you or hand you an answer. Each one interrogates you — pressing on vague claims and refusing to let you off the hook for a weak answer — the way Jason would if he were facilitating in the room. You leave with your own best thinking, not the AI's."

This is a genuine inversion of the default helpful-assistant stance. Where [[Socrates Skill]] refuses to answer in order to *teach*, and [[Matt Pocock — Grill Me, Then Go AFK]]'s grill-me interviews the human to *align*, Cohen's skills refuse to answer in order to *break* the plan — to expose the denial that no amount of self-reflection reaches. The "Scars" article supplies the deepest justification: even highly self-aware people cannot fix their own blind spots alone.

## The Two Skills in Depth

### Rude Q&A: The Constructive Devil's Advocate

> "This attacker asks unfair questions, refuses vague answers, and stays on each point until you give a real answer."

The framing that makes it work is the **heavy bat**:

> "You face attacks harder than reality will bring, so the real thing feels easier. The questions are deliberately rude, but the tone around them stays friendly."

Three honest outcomes are specified up front: a sharper plan with consequences accepted, a list of what you still need to decide, or the conclusion that the idea was wrong. The skill outputs a markdown document recording decisions, reasoning, accepted consequences, and the points where you felt uncomfortable — it's adversarial *interrogation that produces an artifact*, not just a conversation.

### Good Market?: The Seven-Factor Fermi Scorecard

The second skill (`asb-problem`) operationalizes Cohen's article "Excuse me, is there a problem?" into seven multiplying, power-of-ten criteria — **Plausible, Self-Aware, Lucrative, Liquid, Eager (identity), Eager (comparative), Enduring**:

> "Solving a real problem is not enough to build a successful company… Founders often confirm that the problem is real. They build a product that solves it. Many still fail. Five more gates stand between 'a real problem' and 'a viable business model.'"

The score is explicitly directional, and the skill "will not score a generic idea" — it challenges every optimistic number unless evidence backs it, can do light research to check market-size claims, and when the score is negative, pushes toward a narrower niche before conceding the idea isn't viable. Each criterion is grounded in its own article (Product Purgatory for Liquid, willingness-to-pay for Eager, leverage and "Worse, but unique" for comparative differentiation, pricing for Lucrative).

## What Makes This Different

Cohen's skills sit at the intersection of three patterns the wiki has seen separately:

- **Framework-as-skill**, like [[Decision Framework Skill]] (37signals' 38 questions) and [[CEOS (Claude + EOS)]] (EOS) — a practitioner's hard-won domain taxonomy encoded as instruction structure. The "seven criteria" and "Find Your Carol" (ideal-customer) workshops are Cohen's blog distilled into question taxonomies.
- **The AI-interviews-you inversion**, like [[Matt Pocock — Grill Me, Then Go AFK]]'s grill-me — the human is the expert, the AI the questioner.
- **Adversarial stance as the product**, the sharpest thread. Rude Q&A's hostile questioning is the business-domain sibling of the adversarial-verification pattern in [[Load-Bearing Assumptions]] and [[Ways of Checking]] — the recognition that a plan (or a claim) survives contact with reality only if someone whose job is to break it has tried.

The distinctive addition is the **epistemic warrant**: Cohen doesn't say "questions are good coaching," he says *you are structurally incapable of asking yourself the hard questions*, and cites a mechanism (denial, excuse-making) for why. That's what justifies the AI's rudeness as a feature, not a bug.

## Critical Analysis

**What's strong.** The "interrogate, don't answer" principle is the clearest articulation yet of a real design axis for skills — a deceleration tool in a landscape of acceleration tools, as [[Decision Framework Skill]] puts it. The Rude Q&A skill's insistence on naming *three acceptable outcomes* (including "the idea was wrong") is a rare, honest specification of what success looks like. Packaging the seven criteria as multiply-to-a-score Fermi estimates gives the founder a concrete, comparable number instead of another checklist.

**What's fragile.** The seven-factor scorecard multiplies seven subjective power-of-ten estimates into a single "viability verdict." Cohen is honest that it's "a simple directional tool," but a precise-looking number from loose inputs invites false precision — the exact failure [[Not-Knowing (Vaughn Tan)]] warns about when risk tools are applied to genuine unknowns. A founder who treats the score as a verdict rather than a conversation has missed the point the skill itself is trying to make.

**What's assumed.** The skills deliberately won't think for you, which means their value ceiling is the user's own judgment and domain knowledge. That's the point — Cohen wants *your* thinking, not the AI's — but it also means a novice gets interrogated without ever being told the framework's conclusions. The skills transfer method, not answers, which is a ceiling for reach if the user lacks the underlying expertise the questions presume.

**The through-line.** What Cohen's package and the Socratic/grill-me skills converge on is a shared answer to the question "what is an agent for?" Not production of output, but *forcing the human to produce better output*. It's the same instinct as [[Optimizing for Decision Points]] — the agent's job is to surface the decisions that matter and hold the human to them, not to fill them with safe defaults.

## Tags

#tool #pattern #concept #person #claude-code #skills #interrogation #socratic-method #business-strategy #fermi-estimation

## Cross-References

- [[Socrates Skill]] — The same "never gives you the answer" stance, inverted: pedagogy vs. adversarial exposure of denial
- [[Matt Pocock — Grill Me, Then Go AFK]] — The AI-interviews-you inversion, but for code alignment rather than business strategy
- [[Decision Framework Skill]] — Another practitioner's framework turned into a questioning skill; neutral coaching vs. Cohen's hostile interrogation
- [[CEOS (Claude + EOS)]] — The same genre (a founder's framework as a skills package) with the opposite philosophy: run it for you vs. make you think
- [[Load-Bearing Assumptions]] — Adversarial surfacing of unproven claims, the verification-domain cousin of Rude Q&A
- [[Thought Refiner Skill]] — Minimal skills that turn vague input into sharp questions
- [[Claude Code Skills System]] — The infrastructure these skills run on

---

*Sources: [[raw/skills-asmartbear-com]], [[summary/skills-asmartbear-com]]*
*Last updated: 2026-08-21*
