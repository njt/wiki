# Beyond Autonomous Teams — Simon Rohrer (Craft 2026)

Simon Rohrer (Saxo Bank, co-author of *Sooner/Safer/Happier*) spends a Craft 2026 talk demolishing two load-bearing words — "autonomous teams" and "product teams" — and rebuilding org design around **agency within clear boundaries plus coherence**, **value centers** at every level, and **molecular value shapes** whose essential coupling no methodology can dissolve. The last stretch turns the same vocabulary on AI agents: Meta's agency-in-hours metric gets rejected, and context engineering is named the most important thing in the world of AI agents.

---

## What the talk argues

- **"Autonomy" is a container concept.** Autos + nomos = self-law, subject to its own laws — which no team in a complex organization wants. Container concepts (like "sustainability" or "quality") carry whatever meaning you pour into them. German socio-technical writing says *teilautonom* (partially autonomous); Dutch, French and Hungarian say self-steering. He prefers **agency** — capacity to act freely within clear boundaries — paired with **coherence**.
- **"Product" is the wrong unit too.** Scrum's roots are Takeuchi and Nonaka's New Product Development Game — brand-new standalone products. In a complicated organization, what one team delivers is not that; Barclays tried "service," gave up, and called it "a banana." The replacement, from the next book he's co-writing with John Smart: everything is a **value center**, nested all the way up — a flip of cost-center language ("you are there because you are valuable").
- **The shape of your value determines your trade-offs.** A hyperscaler's value comes in tiny atoms (storage, compute, DNS) sellable independently — hence two-pizza teams work. A trading platform sells one thing delivered by 80+ teams around a large shared kernel — orders, prices, risk, portfolios. A universal bank is worse. Some dependencies are accidental (removable with contracts, APIs, decoupling); some are **essential**, and the essential ones are your value: "you're not going to domain driven design away, you're not going to team topologies away" what is essentially coupled.
- **Value designs organizations — fractally.** One step past Conway's Law and Sanchez & Mahoney (1996): not just that org structure shapes systems, but that the shape of value constrains what the organization can look like at every level, including how it can trade agency against coherence.
- **Purpose is what you do.** Stafford Beer: "the purpose of the system is what it does." A hospital A&E where half the patients leave sicker *has the purpose of making people sick* — whatever the mission statement says. True of a team, a department, and you.
- **Strategy and governance flow both ways.** Roger Martin's call-center worker feeds complaints back up into strategy; governance sometimes must come from the top — Novo Nordisk's security incident (per-team platforms and GitHub orgs, no central security policy, Azure DevOps and GitHub access shut down) is the cautionary tale. Platforms are **enabling constraints** — they let you do more, not less; policy is a **governing constraint** for ordered domains; in disorder you align early vs. cohere late.
- **A diagnostic, not a prescription.** Beer's Viable Systems Model modernized into five questions every value center answers at every level: What value am I delivering? How do we coordinate? How do we fit together? What is out there for us? Who are we? Some are "I" questions, some are "we" questions — and they recur fractally.
- **The AI coda.** Meta measures agency as "how long can an AI agent go for without human intervention... They measure agency in hours. I don't think that's what agency is." With five agent tabs open, the value-center questions apply verbatim: what value is each delivering, and how do you make them coherent? Agent boundaries are the harness and its context. Context engineering — sharing skills and context across individual → team → department → organization levels (with Patrick Debois) — is "the most important thing for us to be concentrating on in the world of AI agents." And you will set policy that affects your agents.

## Key quotes

> "Autonomous means self law, subject to its own laws. I don't think that's what teams want."

The etymological gut-punch that frames the whole talk — and the cleanest one-line case for the agency/coherence vocabulary.

> "It doesn't matter what we call them, it's a banana."

On Barclays failing to classify what a team delivers as product or service. Disarming honesty about taxonomy fatigue — and the setup for the rename that follows.

> "You are there because you are valuable. So we're saying squads? No. Value centers."

The cost-center inversion. Psychologically shrewd: it changes what the word *does* to people, not just what it means.

> "Your purpose is making people sick. You should probably consider that."

Stafford Beer's "the purpose of the system is what it does" driven to its uncomfortable conclusion via a failing hospital A&E. The mission statement gets no vote.

> "There is always some coupling left. Always. That's what your value is."

Paraphrasing Vlad Khoninov: the coupling you cannot remove is not waste to be architected away — it is the value itself.

> "They measure agency in hours. I don't think that's what agency is."

On Meta thinning a thick human concept down to a duration metric. The talk's most direct contribution to the agent-era vocabulary.

> "They are now rebuilding and saying, actually we made a mistake, we needed a centralized governance policy."

Novo Nordisk post-incident: the confession that some constraints must be governing, not enabling — security cannot be delegated to autonomous (or agentic) teams.

## Key themes

#concept #org-design #sociotechnical #ai-agents

## Critical analysis

**The demolition is stronger than the rebuild, and Rohrer knows it.** The etymology of "autonomy" is rhetorically effective but slightly motte-and-bailey — nobody granting team autonomy means legal self-law. The honest center comes later, when he admits "I worry that even agency and coherence are still container concepts." He does: "agency" and "coherence" are thinner than what they replace. What saves them is that agency-within-boundaries names something *designable* — the boundary — which autonomy never did. For agent systems the boundary is literally the harness, which is why the frame transfers so cleanly.

**Molecular value shapes is the real idea, and the least developed.** It explains two-pizza teams as a consequence of atomic value (storage and DNS sell independently), not managerial virtue — a materialist constraint on org design that beats fashion arguments. But "take a step back, look for the accidental and the essential" (his Q&A answer) is a slogan, not a method. The talk that tells you your org shape is determined by your value shape never tells you how to see that shape.

**Value centers risks being the next container concept.** A renaming with a genuine psychological pay-flip (valuable, not a cost) and zero failure cases — the Novo Nordisk story argues against too much autonomy, but no case study shows a value center working. The actual content is Beer's five questions, which hold up as a diagnostic precisely because they're questions, not answers.

**Purpose-as-what-it-does is an audit lens masquerading as a steering lens.** It removes the alibi of mission statements — the strongest move in the talk — but if purpose is only what you do, you cannot set one, only notice you had one. Rohrer leans on the line without resolving what you then *do* on Monday.

**The AI coda is the most relevant part of the talk for this wiki and the least delivered.** The Meta critique is correct and useful: hours-without-intervention measures risk tolerance, not capacity to act — it's an autonomy-ladder number wearing agency's clothes. The five-agent-tabs coherence question is exactly the right question for multi-agent work. And the Q&A drops an unnoticed gem: AI made static-analysis standards enforcement cheap enough that teams stopped resisting it — a classic agency/coherence conflict dissolved by making coherence nearly free.

## Related pages

- [[Shaped by Demand — The Power of Fluid Teams]] — Dan North's Craft 2025 talk is Rohrer's nearest neighbour in this wiki: same governing/enabling-constraint toolkit, same suspicion of stable-team orthodoxy. North fixes the demand and lets teams self-assemble; Rohrer fixes the value shape and says the org can only trade agency against coherence within it — Rohrer supplies the constraint North's fluid teams silently presume, and North's "autonomy without alignment is anarchy" is Rohrer's whole thesis said earlier.
- [[Who Does What — Team Topologies for the Agentic Platform]] — Rohrer explicitly refuses the project that note extends: "you're not going to team topologies away" the essential coupling, because the coupling is your value. Wulveryck's platform absorbs cognitive load for agents; Rohrer would still ask what value each unit — human or agent — delivers and how the whole stays coherent.
- [[You Shall Not Pass — Where Developers Draw the Line on AI Autonomy]] — the Microsoft survey measures where developers cap AI autonomy (median L3, produce-under-approval); Rohrer's critique of Meta's agency-in-hours metric is the conceptual counterweight: ladder levels measure permission, not agency, and both sources converge on the boundary as the designable object.
- [[Multi-Agent AI Systems Are Organizations]] — Rohrer's "five agent tabs open — what value is each delivering? how are you making that all coherent?" is the value-center question pointed at agent fleets; it strengthens the orgs-of-agents thesis with real org-design vocabulary (coordination, fit, identity) rather than metaphor.

---
*Sources: [[raw/fd17a41b0eb5af024b552a9cec6f0d8e]], [[summary/fd17a41b0eb5af024b552a9cec6f0d8e]]*
*Last updated: 2026-09-13*
