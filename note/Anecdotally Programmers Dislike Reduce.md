# Anecdotally, Programmers Dislike Reduce

Evan Hahn's short observational post: in his experience, `map` and `filter` pass code review without a murmur while `reduce` reliably draws "this part is hard to read" — a small, honest case study in how readability is judged socially, not formally, and in the gap between an author's preference and his reviewers' tolerance.

---

## The Observation

Hahn uses `map` and `filter` constantly and never gets feedback on them. `reduce`, by contrast, attracts review comments and appears far less often in the wild. He documents five theories, honestly including "I'm wrong and the trend isn't real":

- `reduce` is harder to read
- `reduce` is less familiar
- `reduce` can have worse performance than other options
- `reduce` is less elegant in JavaScript, Python, and Swift — but *not* in Clojure, where he got no such feedback
- selection bias on his part

His response to the pushback is accommodation: swap it out and move on.

---

## Key Quotes

> "Often, when I've submitted a patch with `reduce` inside, I get a comment like, 'this part is hard to read.'"

The interesting word is *often*, not *always*. This isn't a style-guide rule being enforced; it's a statistical tendency in human review feedback. Readability objections arrive as feelings, not citations.

> "In my blissful stint as a Clojure developer, I did not get this feedback."

The strongest clue he offers. The objection isn't to the fold abstraction itself — it's to how awkwardly `reduce` sits in imperative-first languages where it's a foreign idiom. Familiarity, not readability, may be the real variable.

> "I've also noticed this less recently, possibly because code review is less thorough nowadays."

The quiet kicker for 2026: the social mechanism that policed unfamiliar abstractions is weakening, and he names declining review rigor as the likely cause.

---

## Key Themes

- **#concept** — readability as social consensus rather than property of the code
- **#pattern** — idiomacity: the same construct is elegant in one language and objectionable in another
- **#tool** — `reduce`/`map`/`filter` as the canonical readability-Rorschach test

---

## Critical Analysis

This is a blog post about one function, but it captures something review culture usually says only in euphemism: **"hard to read" is frequently a proxy for "not idiomatic here."** Clojure Hahn got no complaints; JavaScript Hahn got plenty. The construct didn't change — the local expectations did. That reframing matters for anyone writing style guidance, because a readability rule that's really a familiarity rule will silently punish good code from other traditions.

The accommodation he describes — cave and swap it out — is exactly the rational individual response to a social signal that isn't worth fighting. Multiply by thousands of reviewers and you get why local idiom ossifies regardless of merit.

The 2026 coda is the part this wiki cares about. If `reduce` complaints are fading *because review is less thorough*, that's a small positive measurement of a much bigger regression: the loosening Hahn notices is the same phenomenon documented at scale in [[The End of Code Review]] and [[Agentic Code Review]] — review volume collapsing as code volume explodes. Hahn's anecdote is the human-scale fossil record of that shift: the reviewer who once flagged your fold no longer has the attention budget to.

It also pairs nicely with [[99 Bottles of OOP]]: Metz argues taste is developed through deliberate seeing of code; Hahn documents what happens when code is judged by reviewers whose taste is calibrated to one language's defaults. And with [[A Convention Is Not a Constraint]] — the `reduce` pushback is convention operating as social pressure, not as any enforceable rule, which is why it yields so easily.

---

*Sources: [[raw/2026-09-13-programmers-dislike-reduce]], [[summary/2026-09-13-programmers-dislike-reduce]]*
*Last updated: 2026-10-11*
