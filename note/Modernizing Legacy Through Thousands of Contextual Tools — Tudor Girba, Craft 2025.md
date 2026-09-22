# Modernizing Legacy Through Thousands of Contextual Tools — Tudor Girba, Craft 2025

Tudor Girba (CEO of feenk, transcribed as "Fink") argues that code reading is the single largest expense in software and the only one nobody has ever tried to optimize, because nobody talks about *how* they read. His fix is the testing transformation replayed on a different axis: automated testing won by compressing arbitrary functionality into composable micro-units with a legible signal, so swap "test" for "tool" and manufacture thousands of small, contextual, domain-specific tools that each answer one question about your system. The evidence is one case study, LifeWare, demoed live — and the gist's own summary section is refreshingly candid about what the talk never answers.

---

## Key Quotes

> "If it's not a subject of conversation, it's not explicit. If it's not explicit, it has never been optimized. And we are talking here about the single largest expense we have in our job."

The thesis. Note what it does and doesn't claim: the "asked thousands of people" poll and the unnamed studies establish that reading dominates, but the chain *unspoken → unexplicit → unoptimized* is asserted, not measured. Still, as framing devices go, this is the right one — it explains why tooling investment goes to writing (editors, agents, CI) and almost never to comprehension.

> "You're where testers were 25 years ago."

The placement that carries the whole talk. Manual regression testers were bottom-left on the Wardley map — manual, genesis — and the processes built on them "went boop, they disappeared" once testing became composable and signal-producing. Girba's bet is that code reading sits at the same coordinate and will fall the same way.

> "My tools are not generic. My tools know about that specific problem."

The line that separates moldable development from generic dashboards. When the CEO's signature changes and 36 tests fail for the same reason, the comparison-failure view knows what a *comparison failure means* in this domain, highlights the pixel diff in the rendered insurance document, and accepts the new expectation across all 36 with one click. Domain-specificity is what makes the view portable into a business conversation — the developer walks into a meeting still inside the IDE.

> "It's not always hypothesis first."

The quiet concession to TDD, in the house that Kent Beck's 2002 book built (LifeWare was its case study). TDD presupposes you already know the assertion; often you need to see the narrative first, explore, then decide. This is the talk's most transferable idea, because it dissolves the false choice between test-first discipline and exploration.

> "I don't go to the tools. Tools come to me. Let me say this again, I don't switch to tools. Tools come to me."

Followed by: "Every time you leave, the I has failed." After a test run spawned an interactive web app *inside* the development environment, the factory-floor analogy does the argument: tools come to where the material is. Rhetorically lovely, and it names a real cost — but see the analysis below for where it now rubs against how agent-era tooling actually works.

> "This use case is ephemeral. I have it now. And so I built enough of a tool to help me do the job."

The Polish-translation tool was assembled in "a couple of days," ran a runtime-instrumented query over one test case, scattered the analysis job over the cluster, and produced snippet rewrites with an accept button — system-wide coverage for a use case that will never recur. The legitimacy of throwaway tooling is the part of this talk most practices still refuse to believe.

> "Never ever let the tool drive your decision. Always start with the question that you want to answer and then pick the tool."

Q&A, on how thousands of tools don't become overwhelming. Combined with the second answer — "you just see a few that made sense in little context" — it is more philosophy than mechanism (see analysis).

> "The real bottleneck is in the time to the interesting question."

The closing insight, and the honest one. His adoption path is: start by shrinking time-to-answer, and the question bottleneck reveals itself. He offers no practice for improving time-to-question beyond "start a conversation."

> "It's literally test redesigning the whole business."

LifeWare's modernization method: take an insurer's data — history of every policy, plus every document ever sent to customers — reverse-engineer a replacement system, and demand that replayed output be pixel-identical to the originals, on their own cost, for two decades. TDD scaled from a unit to an entire business, using the cleanest exterior constraint available.

> "I edited a five minute wait. I cheated a little bit."

The staged-demo admission, delivered cheerfully mid-demo. Credit for honesty; the demo's seamlessness is manufactured, and the gist's summary section notices.

## Key Themes

- #concept **Moldable development** — manufacture thousands of small contextual tools, each answering one question; the named methodology, with the book *Rewilding Software Engineering* (with Simon Worley) free at moldabledevelopment.com.
- #concept **Code reading as the largest unoptimized expense** — more than half of working time, never a subject of conversation, therefore never optimized; he asks for 15 minutes a day of attention on it.
- #pattern **Tests-as-tools template** — composable micro-units producing a legible signal (green/yellow/red, where yellow means "the hypothesis no longer holds"); every other question about a system can be approached the same way.
- #pattern **Comparison-failure workflow** — a domain-aware failure class whose view highlights the pixel diff, survives into business conversations, and supports one-click batch accept across same-cause failures.
- #pattern **Ephemeral tooling** — build enough of the tool to do the job; assembled at development time; no polish expected.
- #pattern **Stakeholder / facilitator split** — anyone with a stake may ask questions; facilitators manufacture answers as micro-tools; the environment composes them into narratives that "grow and compound."
- #pattern **Pixel-identical replay** — rebuilding a system from data alone with document-level identity as the oracle.
- #tool **Glamorous Toolkit** — the free, open-source environment behind the demo, positioned as case study, not product.
- #person Tudor Girba · #person Simon Wardley (whose map frames the talk) · #person Kent Beck (whose 2002 case study is the baseline).

## Critical Analysis

**The analogy is load-bearing, and it mostly holds — up to a point.** Tests won because three properties lined up: composability, a legible signal, and cheap action on the signal (red means fix). Girba generalizes the first two cleanly. The third is weaker for tools: a tool's output is an *answer to a question*, not a pass/fail, so "compression" degrades toward ordinary dashboards unless the domain supplies a crisp comparator. That is why the demo's best moment is the comparison failure — insurance documents have pixel-comparable exteriors — and why the cluster views, good as they are, are closer to well-made observability. The gist flags this as "generality of the magic," and it is the right flag.

**The cold-start hole is the talk's real subject, and it is unaddressed.** A talk titled "modernizing legacy" whose case study is a company that has been running TDD since 2002, never stopped modernizing, and gives every developer thousands of on-demand AWS processors is showing the endgame, not the bootstrap. How do you manufacture view number one on a system with no tests, no instrumentation, and no reflective IDE? The honest title would be "what fully modernized looks like." This matters because the first tool built on real legacy is exactly where the cost question — a black box here, with no headcount or ROI for 2,000 tools — has to get answered.

**The maintenance answer is an analogy doing the work of a mechanism, and it cuts both ways.** Asked how 2,000 tools stay alive, he points back at tests: "we had exactly the same conversations 25 years ago." True — and tests did not become free; they became permanent infrastructure with real upkeep, and teams still drown in flaky ones. "Build tools for managing those tools" is recursion offered as a solution. His own demo is quiet evidence: 454 yellow tests in a 23-year TDD shop, 36 of them one cause. The system works — and lives with a permanent failure backlog.

**The AI-shaped elephant is the strangest omission in a 2025 talk about making code reading cheap.** Not one mention of LLMs. The industry's default answer to "what view would let me see this problem instead?" is now "ask a model" — and Girba's answer, deterministic domain-specific views you can inspect and batch-apply, is a *competing* thesis, not an orthogonal one. [[A New Era for Software Testing]] takes nearly the opposite bet (agents as the QA workforce); Girba betrays no awareness of the contest. Ironically, his described workflow — a facilitator manufacturing thousands of small contextual tools on demand — is precisely what a coding agent is good at supplying, which makes the silence look like a missed synthesis rather than a considered exclusion.

**The Wardley map is set dressing.** Introduced at length, it does almost no analytical work beyond placing reading at bottom-left and splitting the world into "use information" (top) and "extract information" (bottom). Borrowed from [[From Here to There and Everywhere — Simon Wardley, Craft 2025]], where it is the native instrument; here it decorates an argument that would survive without it.

**What is genuinely new is the role split and the question bottleneck.** The stakeholder/facilitator inversion is a real alternative to self-serve analytics: instead of teaching everyone to query, manufacture each answer as a bespoke view. And "the real bottleneck is in the time to the interesting question" inverts the usual tooling economics — most of the industry optimizes answering. That he then offers no practice for improving time-to-question is the most credible sentence in the talk: he names the frontier instead of pretending to own it.

**"Every time you leave, the I has failed" reads differently in the agent era.** For a human, context-switching is real cost, and integration is a fair ideal. But agent-mediated work *is* leaving — the loop's whole value is going elsewhere (running tests, reading clusters, editing files) and returning with a compressed result. A discipline optimized for keeping the human in one environment and pushing views toward them is not obviously the right target when the primary reader of the system is increasingly a program. The reading-expense diagnosis survives; the IDE-centric prescription may not.

**On the source itself:** this is a ytx gist — a machine transcription plus a curated summary. The transcript garbles proper nouns ("Tudor Gurma," "wordly map," "multiple development" for *moldable development*), so quote from the summary's cleaned list where possible. The gist's section 4 ("Unanswered Questions and Omissions") is unusually sharp self-aware criticism — rare enough to be worth citing as a feature of the source, not a flaw.

## Related Pages

- [[From Here to There and Everywhere — Simon Wardley, Craft 2025]] — Wardley retells the same feenk translation story ("we spent the first three weeks building the tool") and makes the same reading-cost claim; this gist is the primary source he compresses into an anecdote, and each speaker wields the other's apparatus: Wardley prescribes composable micro tools, Girba opens with a Wardley map.
- [[Composable Tests]] — Beck's isolation/composition desiderata are the mechanism behind "tests are little tools"; Girba strengthens that framing by making tests the template for all tooling, but inherits its hard part: composition demands design judgment, which is exactly what the stakeholder/facilitator split tries to institutionalize.
- [[Before Reading Code]] — same diagnosis, rival prescription: Piechowski's five generic git commands are the cheap, commodity end of the very map Girba draws, while moldable tools are bespoke per-question views; both agree you should decide what to look at before you read.
- [[Auditing Legacy Rails Codebases]] — the manual-inspection tradition this talk wants to obsolete: the nine audit questions are "digging through some mine to find the nuggets," generic where moldable tools are question-specific; Girba's answer to every one of those questions would be to manufacture a view.

See also [[How SQLite Tests Software]] for another testing-at-absurd-scale identity story, and [[A New Era for Software Testing]] for the agent-side answer to the same expense.

---
*Sources: [[raw/modernizing-legacy-through-thousands-of-contextual-tools-tudor-girba-craft-2025]], [[summary/modernizing-legacy-through-thousands-of-contextual-tools-tudor-girba-craft-2025]]*
*Last updated: 2026-09-13*
