# What Happened to Silicon Valley (O'Reilly)

Tim O'Reilly's 2026 Margaret Levi Lecture is a veteran insider's account of how Silicon Valley moved from creating value to extracting it — told through economic rents, the "supermoney" betting economy, and the hype cycles from Uber to Web3 to the AI data center buildout — and closing with a mechanism-design argument for what the AI economy ought to become instead.

---

## What the lecture argues

O'Reilly opens with the web he thought he was building in 1993: GNN, the first portal and first ad-supported site, designed around *utility* rather than attention-grabbing. He frames this as "harnessing collective intelligence" — Hal Varian's stat that only 6% of Google's search pages carried any monetization is his emblem of value created far in excess of value captured. The web was a public good even when privately owned.

The problem is not villains but the machine. Corporations are, in Charlie Stross's phrase, a "slow AI" whose objective function rewards shareholders and ignores everyone else — so every actor is eventually incentivized toward extraction, no matter their original values. His analytical toolkit is **economic rents**, split in two:

- **Rising tide rents** — Schumpeterian rewards for innovation, often government-seeded (the NSF grant behind Google, loan guarantees for Tesla). Legitimate, and supposed to be competed away.
- **Robber baron rents** — income above competitive rates collected through raw market power, exactly what companies are taught to defend with moats once growth slows. Cory Doctorow's *enshittification* names the transition, which O'Reilly calls close to a law of the technology business cycle.

Then the financial layer: Jerry Goodman's "supermoney." A dollar of Tesla profit was worth ~$600 of market cap; a dollar of Google's, ~$34; the unfinancialized dry cleaner's, $1. Two economies, two rulebooks — and the betting economy is fine in moderation, funding infrastructure ahead of prosperity. The problem is what we bet on. The AI buildout, he argues, is financed against future demand from companies that haven't found product-market fit, so it only needs the hype to recede, not demand to collapse, to fail. AI has also "skipped the youth": extractive training-data enclosure (Barbaro's "AI's original sin," which Levi would call an act of enclosure) means it was enshittified from the beginning, and unlike Google — which pointed you back into the commons 94% of the time — AI offers a replacement for the referral.

His answer is **mechanism design**: "reverse game theory," working backward from a desired outcome to the rules that produce it. Unix accidentally built "an architecture of participation"; MCP, he argues, is doing the same for the agentic web, avoiding a winner-takes-most API market. The closing question: *"What game ought we to be playing?"* — answer: "the game of human flourishing."

## Key quotes

> "We built it. We set its objective function to reward shareholders and ignore everyone else. And then we tell everyone that it is a marvelous self-correcting system. This is an illusion."

The "slow AI" move is the lecture's load-bearing frame: it lets O'Reilly analyze the AI buildout and the corporate form with the same tool. If the market is a machine with a designed objective function, then complaining about tech CEOs is a category error — you have to reprogram the machine.

> "In short, the future cash flows with which data center builders are expecting to pay back the trillions they have borrowed or invested are dependent on those customers' ability to raise their next round at an ever-increasing valuation."

The sharpest structural point in the piece: the AI bubble doesn't even need demand to disappoint, only for hype to recede enough that the refinancing treadmill stalls. "Number go up" is not a side effect; it is the funding model.

> "The commons isn't incidental to AI, it's the material AI is made from."

This is the hinge from critique to prescription. Because models are distillations of collective output, privatizing them is enclosure of the very commons that makes them work — and open protocols (MCP, open weights, open memory) are the mechanism-design answer.

## Themes

#concept (economic rents, supermoney, mechanism design, enshittification) #person (Tim O'Reilly, Charlie Stross, Cory Doctorow, Carlota Perez) #pattern (rising-tide youth → extractive maturity as a business-cycle law) #tool (MCP as accidental market-structure mechanism design)

## Critical take

The lecture is strongest as diagnosis and weakest exactly where O'Reilly admits the difficulty: if the machine's incentive structure forces extraction, then Ries's *Incorruptible* institutions that "you can count on your fingers and toes" is a thin reed. His own logic cuts against his optimism about MCP — a protocol can be an architecture of participation, but participation among whom? The capital behind the model providers already skipped the rising-tide phase; open protocols sit on top of extremely closed weights, and he notes the closed-terms-of-service problem himself. The mechanism-design frame is genuinely useful for reading the current scrum of agent payment and bot-blocking proposals, but he doesn't explain why mechanism design would win this time when it lost to moats every previous cycle. The lecture is more honest as a eulogy with a wishlist than as a plan — still, the rents vocabulary and the two-economies frame are tools worth keeping.

## Relations

- [[Superpowers for Humans (O'Reilly)]] — same author's other side: where this lecture diagnoses the extractive macro-economy of AI, the Vincent conversation shows what O'Reilly's "architecture of participation" looks like at the individual agent level, skills and steering as mechanisms that give users control.
- [[Reliability Is Outside the Model (O'Reilly)]] — companion O'Reilly-published piece; its "deterministic layers enforce, model proposes" thesis is a concrete instance of O'Reilly's mechanism-design demand — guardrails as the designed rules of the game rather than trust in actors.
- [[The Dead Economy Theory]] — complicates this source: both argue the AI economy's cash flows may not close, but the Dead Economy piece treats the circulatory breakdown as already underway rather than a risk to be mechanism-designed away.
- [[Tokens Too Cheap to Meter]] — the unit-economics ground truth under O'Reilly's data-center-financing worry: whether falling inference cost converts to product-market fit is exactly the question his supermoney analysis says the bubble has deferred.
- [[When Your Buyer Is an AI Agent]] — from the same O'Reilly Radar stable; offers the concrete emerging mechanism (agents as buyers) that O'Reilly's closing calls for, the exploratory phase of AI mechanism design in action.

---
*Sources: [[raw/what-happened-to-silicon-valley]], [[summary/what-happened-to-silicon-valley]]*
*Last updated: 2026-11-26*
