# 99 Bottles of OOP

Sandi Metz's practical workbook on writing cost-effective, maintainable object-oriented code. Available in Python, JavaScript, PHP, and Ruby. Uses an extended refactoring of the "99 Bottles of Beer" song to teach OO design not as abstract principles but as a decision-making discipline for every line you write. The 2nd edition is ~105,000 words across 3 new chapters.

---

## Key Quotes

> "It explains the process of writing good code, and teaches you to achieve beautifully programmed ends by way of extremely practical means."

This double claim — beautiful *and* practical — is the book's core argument. Metz rejects the false choice between elegant code and shipping code. The refactoring method she teaches is meant to produce both.

> The Flocking Rules: a structured sequence for safely altering code — find the things that are most alike, make them identical, then extract the difference.

The Flocking Rules are essentially a diffing-and-refactoring algorithm you run in your head. They're one of the few OO techniques that translate directly to how an agent or tool could operate on code: identify sameness, converge, then abstract. It's surprising these haven't been directly implemented in refactoring tools.

> Proper refactoring vs. "rehacktoring"

Metz draws a sharp line: refactoring changes structure without changing behavior, guided by tests. Rehacktoring just rearranges code and hopes. The distinction matters more now than ever — coding agents are natural rehacktorers, rearranging code without the taste to know whether the new arrangement is better.

---

## Key Themes

- **#book** — A programming book that actually works as a workbook
- **#concept** — "Programming aesthetic": taste developed through deliberate practice
- **#pattern** — The Flocking Rules as a structured refactoring algorithm
- **#person** — Sandi Metz as one of the clearest voices in OO education

---

## Critical Analysis

**The Flocking Rules are under-exploited.** They describe a mechanical process — find the most-alike things, make them identical, extract differences — that reads like pseudocode for a refactoring engine. Yet they're taught as a human cognitive practice. In 2026, this is the kind of structured thinking that coding agents could (and probably should) embed directly.

**Metz's "programming aesthetic" is an antidote to vibe coding.** [[Vibe Coding and the Maker Movement]] describes "evaluative anesthesia" — the dopamine of making eclipses the ability to judge what you've made. Metz's entire book is a training regimen for exactly that judgment. She doesn't teach you to *write more code*; she teaches you to *see code differently*.

**The multi-language approach is a flex that proves the point.** Four languages (Python, JS, PHP, Ruby), same ideas, same refactoring — the OO principles genuinely transcend syntax. A book that *needed* to pick a single language to make its case would be admitting the ideas are shallow.

**The book sits at the boundary of two eras.** Published as OO was being challenged by functional approaches and now arriving in a world where AI writes most code, its core insight — that design is a series of moment-to-moment line-level decisions, not a one-time architecture diagram — is more relevant, not less. The question isn't whether agents can write code; it's whether they can develop taste.

**Related pages:** [[Software Engineering Craft]] for the fundamentals that don't change, [[The Mundanity of Excellence]] for excellence as qualitatively different choices, [[Things You're Allowed to Do]] for the confidence to refactor, [[Vibe Coding and the Maker Movement]] for why taste matters more than throughput, and [[Push Ifs Up And Fors Down]] for a distilled rule-of-thumb version of what the Flocking Rules accomplish — converge conditionals at the highest level so branching logic fits on one screen.

---

*Sources: [[summary/99-bottles-of-oop]]*
*Last updated: 2026-06-11*
