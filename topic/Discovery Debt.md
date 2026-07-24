# Discovery Debt

Benedikt Kantus names a silent killer of products: **discovery debt** — the accumulated weight of untested assumptions, unvalidated customer problems, and market insights that teams keep deferring. Unlike technical debt, which shows up in build times and sprint velocity, discovery debt is invisible until the product is "expensively, intricately wrong." Every stakeholder-driven feature built without customer validation, every research session skipped for a deadline, every confident assertion in a room where no one has spoken to a user recently — all accruing interest.

---

## Key Quotes

> "Technical debt slows you down. Discovery debt sends you in the wrong direction entirely."

The core distinction. Technical debt is about speed; discovery debt is about direction. One makes you slower; the other makes everything you build irrelevant. The framing matters because most engineering organizations have language and budget for the first but not the second.

> "Invisible work loses. Every time."

This is the structural diagnosis underneath the whole essay. Shipped features, closed tickets, updated roadmaps — these are legible to the organization. Untested assumptions and conversations never had are invisible. The incentives punish discovery work not because anyone dislikes it but because it can't be *seen*.

> "You're not just wrong. You're expensively, intricately wrong."

The compounding mechanism in one sentence. One unvalidated assumption becomes the foundation for the next decision, then the next. Feature A (untested) → Feature B extends it → Feature C patches the gap. Three levels deep, all wrong, built efficiently by a team that looks productive.

> "Slow is recoverable. Wrong is expensive."

The closing thesis, and the strongest argument for discovery investment. A slow team can optimize. A team building the wrong thing has nothing to show for their speed except a larger sunk-cost argument against changing course.

> "Speed feels like signal. It isn't."

The most dangerous conflation in product development. Velocity metrics tell you how fast you're moving, not whether you're moving toward anything that matters. The team shipping fastest often has the most discovery debt — they're just efficient at building the wrong thing.

> "The question isn't whether you have discovery debt. You do. Every product team does."

Not a platitude — a call to awareness. The first step isn't fixing discovery debt; it's *seeing* it. Most teams don't even have the category.

---

## How to Pay It Down

Kantus offers four habits rather than a process overhaul:

1. **Protect discovery capacity** — ~20% of team time for pure discovery, unconnected to feature milestones. The point isn't the number; it's the *protection*. Discovery time that can be raided for deadlines isn't protected.
2. **Question one assumption per sprint** — one conversation, experiment, or data pull targeting something the team treats as fact. Small enough to actually happen, specific enough to produce evidence.
3. **Talk to users who left or never came** — churned and non-converting users yield harder but more valuable insights than happy customers. Happy customers validate; unhappy ones teach.
4. **Design experiments to prove you wrong** — confirmation-seeking is the default. Invert it. Ask what would need to be true for your assumption to be completely false, then test *that*.

The meta-principle: treat discovery as continuous work, not a phase you completed before development started.

---

## Key Themes

- **#concept** Discovery debt — The accumulated weight of untested assumptions, invisible on team dashboards, that compounds into expensively wrong products. Distinct from technical debt in mechanism, incentive structure, and remedy.
- **#pattern** Visibility asymmetry — Legible work (shipped features) wins resource allocation battles against invisible work (skipped research). The structural force behind discovery debt accumulation, not individual negligence.
- **#concept** Compounding assumptions — Each unvalidated decision becomes the premise for the next. Three features deep on a wrong foundation produces "expensively, intricately wrong" products, not just wrong ones.
- **#pattern** Continuous discovery — Discovery as ongoing habit, not a completed phase. Four tactical habits (protect capacity, question assumptions, talk to detractors, seek disconfirmation) that together shift team rhythm.
- **#concept** Confidence trap — Domain experience creates false certainty about user knowledge. The longer you've worked on a product, the more vulnerable you are — mental models from eighteen months ago are starting points, not substitutes.
- **#person** Benedikt Kantus — Product thinker writing at Leading in Product (Substack). Articulates discovery debt as a named, diagnosable, actionable concept parallel to technical debt.

---

## Critical Analysis

Kantus is right about the diagnosis but soft on the cure. "Protect 20% of time for discovery" is the kind of advice that sounds reasonable in a blog post and evaporates the moment a VP of Sales needs a demo for a $2M deal. The structural forces that create discovery debt — quarterly targets, sales commissions, career incentives tied to shipping — are the same forces that will eat your 20% discovery buffer for breakfast. You don't fix an incentive problem with a time-allocation policy.

The stronger move is to make discovery debt **as legible as technical debt**. If your backlog doesn't have a column for "assumptions awaiting validation," you're not managing discovery debt — you're just feeling bad about it. Technical debt got taken seriously when we started tracking it in the same systems as feature work, with the same visibility to leadership. Discovery debt needs the same treatment: visible, quantified, part of the roadmap conversation, with someone accountable for it.

The article also misses the **confidence trap** in its own prescriptions. Kantus mentions that experienced PMs are most vulnerable ("the longer you've worked on a product, the more you feel you know your users"), but his remedies don't address this directly. A PM with five years on a product won't suddenly start questioning assumptions because you gave them 20% discovery time. You need adversarial process — a [[Load-Bearing Assumptions]] approach where someone whose job it is to be *wrong* about the product forces the conversation. Experience doesn't just give you knowledge; it gives you conviction. Conviction resists evidence.

The technical debt comparison is useful but incomplete. Technical debt has a natural payment mechanism: it slows you down, so you feel the pain directly in your daily work. Discovery debt's pain is displaced — engineers build the wrong thing efficiently, PMs get promoted for shipping, and the pain lands on users and the P&L months later. That's not just a different kind of debt; it's a different kind of **accountability failure**. Nobody feels the pain of their own discovery debt, so nobody pays it down voluntarily.

Where Kantus absolutely nails it: "Speed feels like signal. It isn't." In a culture that worships velocity, the team moving fastest often has the most discovery debt. They're efficient at building the wrong thing. The metric is the problem. Until we measure outcomes (did users actually use it? did it solve the problem?) rather than output (did we ship it? how many story points?), discovery debt will keep compounding.

---

## Related Pages

- [[Load-Bearing Assumptions]] — The operational skill for surfacing unverified claims that a plan depends on; the adversarial process discovery debt needs
- [[Product-Minded Engineers in an AI-Native World]] — Product thinking as engineering discipline, not role; taste as trainable craft
- [[The PM's Playbook for Shipping AI Features]] — Production engineering playbook for PMs; the discovery-debt equivalent in AI feature shipping
- [[The Cost YAGNI Was Never About]] — Build decisions as options pricing; cheap generation amplifies the build-now-regret-later trap
- [[Writing Code vs. Shipping Code]] — The gap between output and outcomes; AI amplifies commits but not releases
- [[Not-Knowing (Vaughn Tan)]] — Four-type uncertainty framework; when risk tools produce false confidence in the face of genuine unknowns
- [[Engineering for Bounded Cognition]] — Human cognitive limits as design constraints; why "just do more discovery" isn't a complete answer
- [[Nicole Forsgren on AI and Developer Productivity]] — Why shipping hasn't gotten faster despite AI; the bottleneck shifted but the discovery debt problem remains
- [[AI for Product Management]] — Van der Merwe's AI-as-sparring-partner system; one concrete tool for the continuous discovery habit
- [[The AI Productivity Paradox]] — Marty Cagan's argument that AI amplifies the output-over-outcomes flaw: discovery debt is the mechanism, and AI makes it compound faster by industrializing the build-now-ask-never pipeline

---

*Sources: [[summary/discovery-debt]]*
*Last updated: 2026-07-05*
