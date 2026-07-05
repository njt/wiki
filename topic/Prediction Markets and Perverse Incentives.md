# Prediction Markets and Perverse Incentives

A 1,606-point HN thread that accidentally became the definitive taxonomy of what's wrong with prediction markets. Triggered by a Times of Israel report about Polymarket users threatening to kill a journalist over an Iran missile story, the discussion maps the full argument space: regulatory capture, perverse incentives, the gambling-vs-information debate, and whether privacy fixes anything.

---

## Key Quotes

> "These prediction markets incentivize the absolute worst in humanity."

Pindab0ter sets the tone. The thread goes further — it's not just that prediction markets *attract* bad actors, it's that they *create* bad actors by aligning financial incentives with harm.

> "Just call it gambling. They aren't 'prediction markets,' they're just gambling."

Scoofy's bluntness gets to the semantic core of the debate. The "prediction market" label is a regulatory hack — Polymarket/Kalshi successfully lobbied the CFTC to classify them as "democratizing futures" rather than gambling. The name is the product.

> "We're talking about gambling as big players paying participants to throw fights, paying referees to call shots, and the players are the real world and the referees are journalists."

Fritzo draws the sharpest distinction in the thread. This isn't addiction gambling — it's market manipulation where the "game" is reality and journalists are the officials you bribe or threaten. The market doesn't *predict* reality — it *competes* with it.

> "No matter what he reported, he would have the other side threatening him."

Awakeasleep captures the journalist's trap. When money sits on both sides of a binary outcome, any reporting that moves the probability becomes a threat to someone's position. Truth becomes a liability.

> "The entire social value rationale is actually what makes prediction markets dangerous / socially bad."

Sam0x17, who proposed privacy-preserving protocols, arrives at the core paradox: the thing that supposedly makes prediction markets valuable (surfacing collective sentiment) is exactly what makes them corrupting. The signal is the weapon.

---

## Key Themes

**#concept: Perverse incentives.** The thread's central finding: prediction markets don't just measure reality — they create incentives to change it. As saalweachter illustrates, to eliminate a rival you bet on their survival (a superficially "good" bet), incentivizing others to harm them. The market structure guarantees misaligned incentives regardless of the bettor's intent.

**#tool: CFTC regulatory capture.** Alephnerd's explanation of how Polymarket successfully argued they're "democratizing futures" rather than operating a gambling platform is a case study in regulatory arbitrage. Backed by YC and Sequoia, they captured a regulator designed for agricultural commodities and bent it to cover assassination-adjacent betting.

**#pattern: The naming-is-the-product defense.** The fight over "prediction market" vs. "gambling" vocabulary is the entire business model. As long as they're called prediction markets, they fall under CFTC rather than state gambling laws. Scoofy's "just call it gambling" is a structural argument disguised as a semantic one.

**#concept: Information as weapon.** Dlenski identifies the paradox: the supposed social value of prediction markets is surfacing aggregated sentiment. But that same signal is what lets gamblers know the target journalist is "costing them money." You can't have the information benefit without the information weapon.

---

## Critical Analysis

The thread is unusually good for HN — 1,600+ points and the quality holds. What makes it work is that it's not a debate between "prediction markets good" and "prediction markets bad" but a finer-grained dissection of *which mechanisms* cause harm and *whether those mechanisms are fixable*.

The privacy proposal (Thread E) is the most revealing failure mode. Sam0x17's suggestion to hide order-book sides is technically clever but misses the point in a way that's instructive: the problem isn't that bettors can *see* odds, it's that they know their *own* stake. A bettor who put $50,000 on "missile strike" doesn't need a public order book to know the journalist's story threatens their position. Janalsncm catches this immediately.

The "just call it gambling" argument is correct but incomplete. Yes, these are zero-sum wagers on uncertain outcomes. But what makes Polymarket different from sports betting isn't *what* you bet on — it's that sports betting can't make the game happen. A bet on the Super Bowl doesn't change who wins. A bet on "will Iran strike by April" creates incentives to *make* Iran strike by April. That's the qualitative difference the gambling analogy obscures.

The real insight buried in the thread is that prediction markets are structurally incapable of being "just information." Every bet is simultaneously a prediction *and* an incentive. You can't decouple them any more than you can observe a quantum system without affecting it. The policy implication is that both the libertarian "let markets bloom" position and the reformist "add privacy" position miss the point — the problem is intrinsic to the structure, not the implementation.

The thread is also a case study in how regulatory capture works in practice: 17 CFR § 40.11 already bans contracts on war and assassination. The law exists. The CFTC simply chooses not to enforce it. The mechanism isn't missing legislation — it's captured enforcement.

---

*Sources: [[summary/polymarket-death-threats-hn]]*
*Last updated: 2026-05-15*
