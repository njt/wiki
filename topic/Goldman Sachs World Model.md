# Goldman Sachs World Model

Goldman Sachs Global Institute argues that "world models" — AI systems that internally simulate physical and social reality rather than just predicting text — are the next architectural leap, and that the investment community is systematically underestimating the compute demands they'll create.

---

## Key Quotes

> "If large language models give AI fluency, world models give it situational awareness."

The report's thesis in nine words. Fluency without grounding is impressive but fragile — this is the Goldman version of the argument that [[The Car Wash Question|LLMs lack common sense]] because they've never encountered reality.

> Current LLMs "learned about the world by reading what humans wrote about it" without "ever encountering reality itself." They "lack the internal sense of the world those patterns describe" and "generate this understanding through second-order interpretation."

The core critique of pure language models. An LLM can describe a glass shattering but has no internal model of weight, trajectory, or consequence. This isn't a scaling problem — it's an architectural one. You can't train your way to physics.

> "World models are not forecasts." They "don't predict the future in any narrow sense; they're meant to reveal plausible futures and expose hidden dynamics."

Important distinction that the breathless coverage will miss. World models are scenario generators, not crystal balls. The value is in understanding the shape of possible outcomes, not picking winners.

> "Competitive advantage might depend as much on who trains the largest model as who builds the most faithful simulations of reality."

Goldman telling its own clients: the LLM scaling race might not be the one that matters. The firm that builds the best simulator of reality wins the next round.

> Simulation creates compute demands "not yet reflected in consensus supply-and-demand forecasts."

Translation: if world models take off, everyone's infrastructure models are wrong. This is the money quote for investors — Goldman is flagging a supply-demand gap in compute before the market has priced it in.

---

## Key Themes

#world-model #AI-architecture #simulation #LLM-limitations #compute-infrastructure #concept

**Two tracks, one idea.** Physical world models teach AI physics through simulation — robots practice millions of digital failures before touching reality. Virtual/social world models populate digital environments with AI agents that have goals and incentives, so "patterns emerge. Markets behave. Organizations respond. Crises cascade." Same principle, different substrate.

**The LeCun bet.** Yann LeCun has been on this for years — his JEPA (Joint-Embedding Predictive Architecture) at AMI Labs is a direct attempt to build world models that learn by predicting representations of reality rather than next tokens. Goldman is effectively blessing his research direction as investable.

**Fei-Fei Li's spatial intelligence.** World Labs, her Stanford-spawned startup, frames the problem as "spatial intelligence" — understanding 3D space, object permanence, and physical causality. Different vocabulary, same problem: how do you give AI an internal model of reality?

**The infrastructure play.** Goldman's real argument is for investors: current AI infrastructure projections assume continued LLM scaling. World models need "purpose-built data pipelines, synthetic data generators, and physics-based engines" — different compute patterns, different bottlenecks, different winners. [[KV Cache Locality]] and [[How AI Labs Are Solving the Power Crisis]] are the current infrastructure conversation; world models add a new dimension.

---

## Critical Analysis

**The argument Goldman is actually making** is not about AI research — it's about investment allocation. A Goldman Sachs Global Institute report is a product designed to move institutional money. When they say "competitive advantage might depend on who builds the most faithful simulations of reality," they're not doing philosophy — they're telling clients where to put capital. Read accordingly.

**The report is right that LLMs don't have world models**, but it undersells how far pure language models can go without them. Humans also learn plenty about reality through text — nobody has personally verified that water freezes at 0°C, yet we all use that fact reliably. The line between "second-order interpretation" and "understanding" is fuzzier than the report admits. [[A Non-Anthropomorphized View of LLMs]] takes the opposite extreme — LLMs are just functions through ℝⁿ — but the truth is messier than either framing.

**The two-tracks framework is useful but incomplete.** Physical and social world models are presented as parallel efforts, but the hard problem is their intersection: AI that understands that a supply chain disruption (social) changes what can be physically delivered (physical). Goldman waves at this with "crises cascade" but doesn't engage with the integration challenge.

**World models have their own scaling problems.** Simulating reality at useful fidelity requires modeling the right things at the right resolution. The combinatorial explosion is worse than language modeling — you can't just throw more GPUs at it. The report mentions "purpose-built data pipelines" but doesn't wrestle with the fact that simulation fidelity is a design problem, not a compute problem. See [[2026 Global Intelligence Crisis]] for the S-curve argument that infrastructure constraints bite harder than AI enthusiasts admit.

**The LeCun/Fei-Fei namedrop is both informative and strategic.** By anchoring to recognizable AI luminaries, Goldman makes the investment thesis feel grounded in research rather than speculation. But LeCun has been on the world-model train for years without a breakout product, and World Labs is still early. The gap between "important research direction" and "investable thesis" is Goldman's product — they're selling the bridge.

**Bottom line:** The report is worth reading for the taxonomy (physical vs. social world models) and the investment signal (compute demand not yet priced in). Don't mistake it for neutral research — it's a market-making document from a market-making firm. The framing that world models are "AI's missing link" is a useful organizing concept, but the road from current LLMs to useful world models is longer and harder than the report implies.

---

## See Also

- [[The Car Wash Question]] — the cleanest demonstration of what LLMs lack: common-sense inference about unstated real-world facts
- [[A Non-Anthropomorphized View of LLMs]] — Flake's mathematical framing of LLMs as functions; world models are the counterargument that internal structure matters
- [[2026 Global Intelligence Crisis]] — infrastructure constraints and S-curves; world models add a new dimension to the compute-demand question
- [[How AI Labs Are Solving the Power Crisis]] — the physical infrastructure that world model compute demands would stress
- [[KV Cache Locality]] — current AI infrastructure optimization; world models may need entirely different optimization patterns
- [[Context Is Not Learning]] — the distinction between in-context reasoning and learned internal models; world models aim for the latter
- [[The Future of Everything is Lies I Guess]] — Kingsbury's catalog of LLM harms; world models could mitigate some (grounding in reality) and amplify others (simulation as manipulation)
- [[Interfaze (Model Architecture)]] — hybrid architecture routing tasks to specialized subnetworks; a different approach to the same "LLMs aren't enough" problem
- [[Where the Goblins Came From]] — what happens when model internals encode things nobody intended; world models would have their own goblins

---
*Sources: [[summary/goldman-sachs-world-model]]*
*Last updated: 2026-05-22*
