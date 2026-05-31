# You Can Just Say It

Caleb Gross argues that the standard defenses of human creative value in the age of AI — "AI can't do X," "humans do it better," "the style is different" — all depend on a narrowing capability gap that may not hold. His alternative: just say humans are valuable, no qualifiers needed. He then builds a precise definition of "AI slop" as form without discernible intent, and argues the pathology of generative AI is how easily it produces substantial form with minimally applied intent.

---

## The Central Move

Gross's first point is simple and devastating: every "but AI can't do *this*" argument is a bet on a temporary capability gap. When the gap closes — and it keeps closing — the argument collapses. The people making these arguments are conceding the premise that human value needs to be justified by superiority over machines.

His counter is a refusal to play that game at all:

> "Humans are valuable."

No hedging, no qualifier, no "because we're better at X." Just the assertion. He calls this a position "not conditional on a point-in-time snapshot of the leading frontier model's score."

This is either profound or tautological depending on your priors. Gross knows this — the footnotes (Genesis 1:27, Pope Leo XIV's *Magnifica Humanitas*) gesture at the theological framework he's operating from without requiring the reader to share it. The move is: your value doesn't need to be proved; it needs to be asserted.

## Intent and Form

The essay's second half is stronger than its first. Gross proposes a two-component model of creative quality:

- **Intent** — what the creator meant to express
- **Material form** — what actually exists in the world

Creation is "the distillation of intent into form." The human iteratively shapes the artifact until it matches their mental model. The peculiar feature of generative AI is that it **inverts this**: it "can produce substantial form with minimally applied intent." You can get a perfectly formatted, grammatically correct, stylistically coherent output without ever having a clear idea of what you wanted to say.

This is a better definition of "AI slop" than anything else I've read:

> "AI slop" describes artifacts where the intent behind the form is hard to discern.

Crucially, Gross notes humans produce slop too — AI just "lowered the entry barrier for creating intentless form." The problem isn't the tool; it's that the tool makes it too easy to skip the hard part (figuring out what you mean) and jump straight to the easy part (producing something that looks finished).

## The Best Line in the Essay

> "If you're going to use an LLM to write me an email, I'd much rather you just send me the prompt; at least then I'd have an idea of what you actually meant to say."

This is Tom Hudson, quoted by Gross, and it's the most practical takeaway in the piece. For prose — unlike visual art or music — a well-developed prompt is often already "near the intended form." The prompt *is* the communication. Wrapping it in AI-generated prose doesn't add clarity; it obscures intent.

This connects directly to [[Intent Is the Interface]] — the prompt is the intent layer, the polished email is the rendered form. If you can skip the rendering without losing meaning, you should.

## Key Quotes

> "The pathology of generative AI is that it too easily allows substantial form without discernible intent. That mistake is harder to make when creating by hand."

The essay's thesis, compressed. When you write by hand, producing substantial form *requires* wrestling with intent — you can't cheat. The constraint is the safeguard. Removing the constraint removes the safeguard.

> "Many defenses of creative output focus too much on form at the expense of intent."

Gross's diagnosis of why the standard AI-vs-human debates feel hollow. Everyone's arguing about the artifact when the real question is what the artifact encodes.

## Key Themes

- **#concept Intent-over-form** — The quality of a creative artifact is determined by the discernible intent behind its form, not the form alone. AI makes it easy to produce form-first, intent-never output.

- **#concept AI slop as intentlessness** — The cleanest definition of "AI slop" in the wiki. Not "bad writing" or "weird style" — those are symptoms. The disease is form without discernible intent. This reframes slop from an aesthetic judgment to a structural one.

- **#pattern The prompt as the artifact** — Hudson's insight: for text communication, the prompt may already *be* the intended form. Rendering it through an LLM adds polish but subtracts signal. This is the prose equivalent of [[The Plan Is the Program]] — the plan (prompt) is closer to intent than the output it generates.

- **#concept Human value as axiom, not theorem** — Gross's philosophical move: stop trying to *prove* human value by pointing to things AI can't do. Assert it. The capability-gap defense is strategically fragile because the gap keeps shrinking. The assertion defense is "not conditional on a point-in-time snapshot of the leading frontier model's score."

## Critical Analysis

**The first half is weaker than the second.** "Humans are valuable" is either the most important thing anyone's said or a content-free assertion, and Gross doesn't do the work to bridge that gap. The theological footnotes suggest he knows this — he's gesturing at a framework that would support the claim without building it. Fair enough for a blog post, but the essay's first argument is a mood, not an argument.

**The intent/form framework is the real contribution.** This is a genuinely useful lens that applies far beyond AI writing. You can use it to evaluate any creative artifact — code, design, music, conversation — by asking: how much of the form is accounted for by discernible intent? When the answer is "very little," you're looking at slop, regardless of who or what produced it.

**The Hudson quote is the essay's most subversive idea and Gross underplays it.** If "just send me the prompt" is better communication than sending AI-polished prose, then a large fraction of AI writing use cases evaporate. Not the ones where people are genuinely using AI to clarify their own thinking — those use cases are about *developing* intent, not obscuring it. But the "make this sound professional" use case, which is probably 80% of business AI writing, is exposed as actively harmful. You're not adding information; you're stripping the signal that tells the reader what you actually meant.

**This pairs with** [[Why Does AI Write Like That]] (Kriss's catalogue of AI prose tics is what "form without intent" looks like at the sentence level), [[Creative Firewall]] (Sundar's Trojan Prompts are prompts with maximally dense intent — the opposite of slop), [[Intent Is the Interface]] (Marc's thesis that interfaces should be derived from intent rather than designed by hand), [[The Plan Is the Program]] (Angert's aphorism — the plan encodes intent; the code is a rendering), [[Automatic Programming]] (antirez's bright line: programming is automatic, vision is not), [[Write Only Code]] (the Slop Radius as the distance intentless code travels before detection), [[Specifications as the Product]] (specs encode intent; code is disposable form), [[Breaking the Spell of Vibe Coding]] (Thomas's diagnosis of evaluative anesthesia — the inability to judge whether form actually encodes intent), and [[Talking to Transformers]] (domain language as compression — intent-dense prompts are compressed intent).

---

*Sources: [[raw/you-can-just-say-it]]*
*Last updated: 2026-05-31*
