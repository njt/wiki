# Writing Code vs. Shipping Code

The first rigorous empirical study of how AI coding productivity gains cascade (and decay) through the software production hierarchy. Using 100K+ GitHub developers and AI telemetry, Demirer, Musolff, and Yang find that while autonomous agents increase commit activity by 180%, that gain attenuates to just 30% at the level of actual software releases. The bottleneck isn't the AI — it's everything else. This is the most important paper on AI and software engineering in 2026, and it should reframe how we talk about "10x developers."

---

## Key Findings

**The generation stack matters.** Cumulative effects on coding activity (commits):

| Tool generation | Effect on commits |
|---|---|
| Autocomplete (e.g., Copilot) | +40% |
| Interactive coding agents (e.g., Cursor, Claude Code) | +140% |
| Autonomous coding agents | +180% |

Each generation compounds on the last. Autocomplete helps you type; interactive agents help you think; autonomous agents can build without you.

**But then it all falls apart.** The 180% gain at the commit level attenuates sharply:

> Commits: +180% → Projects: +50% → Releases: +30%

Translation: AI lets you write a lot more code, but most of it never ships. The production hierarchy is a series of filters, and each filter removes AI-generated output at a higher rate than human-generated output.

**The weak-link hypothesis.** The paper's core theoretical contribution: AI productivity gains are bottled up by whatever step in the production chain is hardest to automate. Writing code got easy. Code review, integration, testing, deployment, user research, and stakeholder alignment did not.

> The strong productivity gains from AI are attenuated by human bottlenecks in the production chain, with an estimated elasticity of substitution of 0.25 between AI and human effort, which indicates strong complementarities.

An elasticity of 0.25 means AI and humans are *complements*, not substitutes. More AI doesn't let you use less human effort — it makes human effort *more valuable*. This is the opposite of the substitution narrative driving AI anxiety.

**The marketplace test.** Across four major app marketplaces: moderate increase in *new apps*, zero increase in *total usage*. More software is being created, but users aren't using more of it. The supply curve shifted right while the demand curve stayed put. This is a classic Jevons-paradox-in-waiting scenario: cheaper code doesn't automatically create more value.

---

## Why This Matters

This paper kills several narratives at once:

1. **"AI will replace developers."** No — elasticity of 0.25. Strong complements, not substitutes. The bottleneck shifts to what humans do, making human judgment *more* scarce and valuable, not less.

2. **"10x developers."** The 180% commit gain sounds like 2.8x. But commits aren't the output. At the level of *shipped software*, the gain is 1.3x. Real, but not revolutionary. The "10x" rhetoric confuses an input metric with the output that matters.

3. **"More code = more value."** The marketplace data is brutal: more apps, same usage. The limiting factor isn't production capacity — it's *demand for software*. We're already making more than people want to use.

4. **"The bottleneck is AI capability."** It isn't. The bottleneck is the rest of the production chain — the human coordination, review, and decision-making that AI doesn't touch. Making models smarter won't fix this; it'll just pile up more unshipped code.

---

## Key Quotes

> "Autocomplete, interactive coding agents, and autonomous coding agents each significantly increase coding activity ('commits'), with respective cumulative effects of 40%, 140%, and 180%."

The generational effect sizes are real and large. But commits are an input metric — the paper's entire contribution is showing what happens when you measure *output* instead.

> "These gains, however, attenuate sharply across the production hierarchy: the 180% cumulative effect falls to 50% for the number of projects, and to 30% for actual releases."

This is the money quote. The attenuation is the finding. Every step away from raw code generation and toward shipped value strips away most of the AI gain.

> "An estimated elasticity of substitution of 0.25 between AI and human effort, which indicates strong complementarities."

This number should be quoted everywhere. It's the empirical rebuttal to substitution-panic. AI makes human effort *more* productive, not redundant. The people who benefit most are the ones who already know what to build and how to verify it.

> "We further confirm these results across four major app marketplaces, finding a moderate increase in the number of new apps but no increase in total usage."

Damning. The app stores are filling up with AI-generated software nobody wants. Supply without demand. The paper's quietest finding is its most devastating: we're automating the wrong part of the value chain.

---

## Key Themes

#concept **Weak-link hypothesis** — In multi-stage production, overall output is constrained by the least automatable step. AI accelerates code generation; everything downstream becomes the bottleneck.

#concept **Elasticity of substitution** — 0.25. AI and human effort are strong complements. More AI → human judgment more valuable, not less. This is the empirical nail in the "AI replaces developers" coffin.

#pattern **Production hierarchy attenuation** — The further you get from raw code, the more AI gains shrink. Commits → PRs → merges → releases → adoption. Each step filters AI output harder. Measure at the wrong level and you're lying to yourself.

#tool **Matched event study design** — 100K+ GitHub developers, AI telemetry, marketplace data. The methodology is as important as the findings: this is what rigorous measurement looks like vs. vibes-based productivity claims.

---

## Critical Analysis

**The paper's strength is also its limitation.** It measures what's measurable — commits, projects, releases, app store listings. But the highest-leverage effects of AI coding might be invisible to these metrics: architectural decisions made better, bugs *not* written, refactors that *didn't* need to happen. The paper can only say what happened to observable output; it can't measure what *would have happened* without AI. If AI prevents bad decisions that would have cost months of rework, none of that shows up in the data.

**The marketplace finding is undersold.** "No increase in total usage" across four app stores is a staggering result that gets buried in the abstract. If AI lets us produce more software but users don't use more software, the economic value of AI coding tools accrues to the *consumers* of existing software (who get more features) and the *tool vendors* (who collect subscription revenue), not to new software creators. This has profound implications for the startup economics that the venture industry is betting on.

**The weak-link hypothesis explains why structural approaches beat model upgrades.** [[Structural Backpressure Beats Smarter Agents]] argued that process constraints outperform smarter models. Demirer et al. provide the theoretical framework for *why*: the weak links aren't in the AI. They're in code review, testing, deployment, coordination. Throwing a better model at a process bottleneck is like putting a faster engine on a car with square wheels.

**The policy implication is uncomfortable.** If AI and human effort are strong complements (0.25 elasticity), then the developers who benefit most are the ones with the most judgment, taste, and domain knowledge — i.e., senior engineers. Junior engineers who can only produce raw code without the surrounding skills get commoditized. The inequality dynamics are baked into the complementarity finding, though the authors don't dwell on it.

**The paper doesn't engage with the dark factory thesis.** [[The Dark Factory is a DOT File]] and [[StrongDM Factory Techniques]] argue that the durable artifact is the pipeline spec, not the code. If that's right, then measuring "releases" still isn't measuring the right thing — the valuable output might be *process knowledge* that gets encoded in specs and workflows. A follow-up study measuring spec quality, pipeline reuse, and process improvement would tell a different story.

**The NBER version vs. the SSRN version is worth noting.** Posted two days apart, nearly identical. The NBER stamp signals that economists take this seriously. Expect this to be cited in every AI-and-labor paper for the next five years.

---

## Connections

- [[AI Coding Tools Create More Bugs Than They Fix]] — Mobb's finding that AI-generated code breaks security controls complements Demirer's attenuation pattern: the code that *does* ship may be worse
- [[Structural Backpressure Beats Smarter Agents]] — Process beats intelligence because the weak links aren't in the model
- [[The Mythical Agent-Month]] — McKinney's observation that agents generate *new* accidental complexity that obscures essential structure; Demirer provides the empirical evidence
- [[Five Levels from Spicy Autocomplete to the Dark Software Factory]] — Shapiro's generational framework maps directly onto Demirer's three tool categories
- [[Probabilistic Engineering and the 24-7 Employee]] — Davis on the Jevons paradox of infinite code: Demirer shows the supply increase is real; Davis explains why demand hasn't caught up
- [[Specifications as the Product]] — If code is cheap and disposable, the durable artifact is the spec. Demirer shows *why* code is cheap; the spec thesis shows what to do about it
- [[Agent Coding Workflow]] — The bottleneck was never typing speed. Now we have the numbers
- [[Coding Agents and Complexity Budgets]] — Robinson's thesis that abstractions consume a budget agents can't spend; Demirer's weak-link hypothesis formalizes this
- [[Compound Engineering]] — The Compound step is where human judgment integrates AI output into something that actually improves the system; Demirer shows this step is where the value lives
- [[AI Killing B2B SaaS]] — The marketplace finding (more apps, same usage) directly challenges the "AI kills SaaS" narrative
- [[The Road Runner Economy]] — Productivity without throughput; Demirer provides the mechanism
- [[Slowing Down in the Age of Coding Agents]] — The gap between generating and shipping, now quantified
- [[Feedback Loop is All You Need]] — Verification, not generation, is the binding constraint

---

*Sources: [[summary/writing-code-vs-shipping-code]]*
*Last updated: 2026-06-05*
