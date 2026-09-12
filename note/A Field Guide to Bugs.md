# A Field Guide to Bugs

Stephen Diehl's poetic taxonomy of 30+ software bug species, stretching from the classical (Bohrbug, Heisenbug, Off-By-One) through the contemporary (Hallucination Bug, Vibe Coding Bug, Recursive Fine-Tuning Bug) to the cosmological (Heat Death Heisenbug, The Omega Bug). The piece is half CS folklore anthology, half literary performance — the joke density and prose quality make it the best debugging essay in years.

---

## Key Quotes

> "Attach a debugger and the bug evaporates."

The Heisenbug in one sentence. This is the experience that gives senior engineers their thousand-yard stare.

> "Its corpses litter the codebase in such density that you can use them as paving stones."

On the Off-By-One. The specificity of the image makes the point better than any statistic could.

> "The most depressing debugging experience is not finding the bug in the code. It is finding the bug in the file, confirming the file is correct, and then discovering the lie is in a README two directories up."

The Comment Lie entry. This is the line that will make every senior engineer stop and stare at the wall for a moment.

> "The human mind, left to itself, silently interpolates state it has not actually verified."

The mechanism behind the Rubber Duck Bug, and arguably the mechanism behind most debugging. The duck is externalized attention.

> "The test suite cannot catch it because the test suite was designed by the cognitive process that produced the bug."

On the Hallucination Bug. This is the deepest critique of AI-generated code paired with AI-generated tests — the error and its validator share an epistemic origin.

> "It is what happens when politeness becomes pathological."

The Livelock, and a sentence that accidentally describes half of organizational dysfunction.

> "The word did not find the thing, the word created the thing."

The Omega Bug's thesis: taxonomy is not discovery, it's reproduction. Naming a bug gives it form.

## Key Themes

#debugging #pattern #concept #humor

**Bugs have a real ontology.** Diehl opens with the observation that engineers independently converge on identical taxonomies — Bohrbug, Heisenbug, race condition — and treats this as evidence the categories aren't arbitrary. The field-guide form is a joke that's also an argument.

**Debugging is epistemology.** The Schrödinbug (comes into existence when observed), the Heisenbug (vanishes when observed), the Rubber Duck Bug (dissolves when narrated) — these aren't just bug types, they're claims about how knowledge of software systems is constructed and what happens when it's incomplete.

**The LLM era creates new failure modes, not just more failures.** The Hallucination Bug, Vibe Coding Bug, and Recursive Fine-Tuning Bug form a trilogy about AI-generated wrongness: errors that share DNA with their validators, errors that emerge from accumulated refinement rather than any single mistake, errors that compound across model generations. These are qualitatively new.

**The best debugging writing is literature.** Diehl's prose — "the desperate dignity of a Victorian consumptive" for a memory-leaking process, "polite, professional fury" for the YAML bug Slack message, the Omega Bug as cosmic horror — treats debugging not as technical writing but as a literary genre. This approach is more effective at conveying the *experience* of debugging than any technical manual.

## Critical Analysis

The piece is genuinely excellent, but it's worth naming what it is and isn't.

**What it is:** A work of computational folklore. Like the Jargon File or the Hacker's Dictionary, it's collecting and naming shared experiences that every practitioner recognizes but few have articulated. The humor isn't decoration — it's the mechanism. The jokes carry the insight.

**What it isn't:** A debugging manual. Nobody will fix a bug by looking it up in this guide. The taxonomy is descriptive, not operational. That's not a flaw — it's the genre. Complaining this isn't actionable is like complaining Moby-Dick won't help you catch a whale.

**The Omega Bug ending is the right move.** A taxonomy piece like this has to either end modestly ("these are just patterns, use with humility") or go full Borges. Diehl chose Borges, and the piece is better for it. The recursive self-reference — the bug that's read the entry, the reader as vector — turns the field guide from a catalog into a thought experiment about what taxonomies do to the things they classify.

**The LLM trilogy is the most important part.** The Hallucination Bug, Vibe Coding Bug, and Recursive Fine-Tuning Bug entries aren't jokes — they're the sharpest diagnosis I've read of AI-generated software failure modes. "The test suite was designed by the cognitive process that produced the bug" should be a poster in every coding agent office.

**Gaps worth noting:** No mention of the "works in staging, fails in prod" configuration drift bug as its own species (it gets subsumed under It Works On My Machine). No treatment of timezone bugs as a category (distributed across Off-By-One and Yuletide). And the classical species section is thinner on actual debugging strategy than it could be — the Heisenbug entry doesn't mention the standard technique of adding logging in a binary search pattern.

The piece belongs in the same category as [[Better Error Messages]] and [[Elements of Code]] — writing that treats software failure not as an embarrassment to be minimized but as a phenomenon worth understanding on its own terms.

Cross-links: The Hallucination Bug and Vibe Coding Bug connect directly to [[AI Coding Tools Create More Bugs Than They Fix]], [[The Cult of Vibe Coding Is Insane]], [[Breaking the Spell of Vibe Coding]], and [[Vibe Coding and the Maker Movement]]. The Specification Bug is the dark twin of [[The Coming Need for Formal Specification]]. The Comment Lie is documentation-as-liability, connecting to [[Write Only Code]] and [[Cognitive Debt]]. The Rubber Duck Bug is literally [[Duck, Duck, Duck! (IDEO)]]. The Omega Bug's taxonomy-creates-reality thesis echoes [[Where the Goblins Came From]]. The general theme of bugs as inherent to complex systems connects to [[Nobody Knows How Large Software Projects Work]] and [[Probabilistic Engineering and the 24-7 Employee]].

---
*Sources: [[summary/field-guide-to-bugs]]*
*Last updated: 2026-05-22*
