# We're Always Limited by Attention

Kadhir Manickam's short essay argues that AI-augmented engineering speed is the wrong thing to optimise: the binding constraint everywhere is human attention — both the team's and the buyer's — so the whole business must move at one pace, and the point of AI is to fund attention for the bets that matter, not to generate more PRs.

---

## What it argues

The piece opens with a thought experiment: suppose you could release ten brand-new features every week, in perpetuity. Is that good? Manickam, "someone who's been building product for a decade," says hell no — because most platforms run on one or two core features that drive adoption, and spend years hunting for the next adjacent bet. Ten-a-week means you've stopped doing the hard work of understanding why users come and where your value actually is.

The consumer-side mirror: would you even notice a favourite product shipping new features weekly? No — you keep doing what you're comfortable with because you lack the capacity to learn new things constantly. Attention, not feature supply, is the scarce resource on both sides of the transaction. A focused competitor that nails a few things for a well-understood user beats a feature firehose.

His conclusion is organisational: the whole business must speed up together. A feature that drives business outcomes requires validation, iteration, user conversion, new-user capture, pricing, legal — "anything and everything." The dumb rebuttal ("just put more AI in there") fails because "this isn't a technical problem. This is a business and process problem." And we all know not to apply technical solutions to non-technical problems... right?

Finally, he diagnoses the obsession itself: people fixate on engineering speed because it's the problem with the shortest feedback loop. "Why grapple with the hard, nebulous problem of moving the needle in a business when you can focus instead on generating more PRs?" The remedy: estimate how much attention your potential users can give, take high-conviction bets to maximise the ROI of that attention, and backtrack from there to what every department must deliver — "that is the pace at which the business can actually run."

---

## Key quotes

> "Honestly, my reaction as someone who's been building product for a decade now is hell no!"

The hypothetical does real work. It converts a vague "AI makes shipping faster" claim into a concrete absurdity, which is the whole rhetorical engine of the piece.

> "Whether you are a business or a potential buyer or consumer of a product, you are constrained by how much attention you are willing to give to it."

The claim's strength is its symmetry — attention binds producers and consumers alike, so the constraint can't be out-raced from one side.

> "This isn't a technical problem. This is a business and process problem... And we all understand not to apply technical solutions to non-technical problems, right? Right?!"

The sarcastic triple-question-mark is aimed squarely at the AI industry's reflex, and it lands.

> "Why grapple with the hard, nebulous problem of moving the needle in a business when you can focus instead on generating more PRs? That's a simpler problem with a shorter feedback loop."

This is the best paragraph in the piece: it explains the obsession it criticises rather than just mocking it. More PRs = more $$ is not stupidity, it's metric-availability bias.

---

## Themes

#concept (attention as the binding constraint) #pattern (whole-system pacing) #comparison (engineering speed vs. business value)

## Analysis

The essay is thin on mechanism — "figure out how to cleanly spend attention" is easier said than done, and the piece offers no framework for identifying high-conviction bets beyond instinct. It also underplays a real counter-argument: attention constraints are partly solvable by better product design (defaults, agents, automation), which is arguably exactly what AI is for on the consume side. But the core claim is correct and increasingly corroborated: if code is nearly free, the scarce inputs are judgment and attention, and optimising the free thing is a category error.

It's also a useful corrective to the current discourse from someone with product-side rather than tooling-side credibility. Most "engineering is faster now" arguments treat shipping as the product; Manickam insists the business is the product.

## Related pages

This is a product-builder's folk version of [[Modeling Software's Business Impact (James Shore)]], which makes the same "customer attention is the AI-era bottleneck" argument with a formal NPV-and-prioritisation apparatus — the essay supplies the practitioner's intuition, Shore the model. It also complicates [[The Harness Is the Company]], which places human attention at the top of the harness; Manickam would agree, but pushes the same constraint one level out to the *customer's* attention, which the harness essay doesn't touch. And it gives a plain-language explanation for the phenomenon [[AI in the Firm — Bottlenecks in Software Production]] documents empirically — the 30% coding gain that never reaches Jira output — except Manickam locates the bottleneck in the business, not in code review. Finally, [[The Cathedral, the Bazaar, and the Winchester Mystery House]] names "attention" as the new bottleneck at the developer level; this essay makes the same move at the level of the whole firm.

---
*Sources: [[raw/limited-by-attention]], [[summary/limited-by-attention]]*
*Last updated: 2026-10-18*
