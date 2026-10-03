# How Do We Value Code In A World Of Free Code?

jerf's open-ended essay arguing that when generating code trends toward free, the scarcest inputs become attention (finite AI cognition) and exposure to reality (testing in the field). Value migrates from produced code to code that is cheap to understand and battle-tested — and the SaaS moat widens rather than closes.

---

The essay starts from two unrelated problems that turn out to overlap: how a professional programmer should think about the value of code when line production is cheapening toward zero, and how to moderate /r/golang against a flood of AI-generated side projects. The shared answer: judge code by what it costs to process and how much reality it has survived, not by who or what wrote it.

## Key Quotes

> A capability of your software stack is an asset. A line of code is a liability.

The essay's ledger metaphor: an AI that rapidly produces capabilities produces assets; an AI that rapidly produces lines of code produces liabilities. The trouble comes from mis-accounting — putting costs in the benefits column. This is a direct strike at the management culture about to rediscover LOC metrics because "the chart goes Number Go Up."

> AIs will forever in this universe be finite... A finite AI will always do a better job with code that consumes fewer resources rather than more.

The finitude argument is the load-bearing move: even a superhuman-but-finite model pays cognition proportional to code volume, so terse, idiomatic, invariant-preserving code is more valuable *to the AI itself* — not just to humans. The appendix of writing-for-AI-style arguments that currently dominate the wiki.

> I have not had much luck getting AIs to produce code on its own that creates or maintains these invariants... There is apparently no problem that can't be solved by spraying more code, not even the problem of having too much code.

jerf's concrete observation: AIs crack open validated types, redundantly re-check impossible cases, and "spray" point fixes — the firefighter-foam anti-pattern — but at machine speed. Notably the failure mode isn't hallucination; it's that high-quality code isn't well-represented in the training set.

> It doesn't matter how smart AI models get... there is simply no way that a code base that was called into existence ten minutes ago can be *tested against the real world*.

The second pillar: value is real-world exposure, not authorship. A 100% AI codebase with a trillion proxied requests beats a hand-crafted one tested only against telnet. This dissolves the "how much AI was used?" rubric as both tempting and increasingly irrelevant.

> By Amdahl's law, removing all the coding time just means all the other aspects of dealing with a system turn into 100% of the costs.

Why SaaS survives: businesses want their problems to *go away*, not bespoke systems whose problems are theirs. If everyone can summon a million lines over lunch, everyone also lives in a world complicated by everyone else's summoned systems (his HOA-with-tax-authority thought experiment) — and the gap between heavily-field-tested SaaS and instant bespoke code *grows*.

## Themes

#concept — code as liability, cognitive cost as the metric of quality, testing-against-reality as the true value function #pattern — invariant-bearing types as the code AIs can't write but can exploit #tool — none; this is an argument, not a product

## Analysis

This is one of the better attempts to answer the question the wiki circles constantly: what is worth having an agent produce? jerf's contribution is to split "value" into two axes the rest of the discussion tends to conflate — *cognitive cost of the artifact* and *accumulated exposure to reality*. The first axis connects to refactoring economics; the second is essentially the accountability argument in different clothes: what can't be summoned can't be trusted.

Where the essay is strongest: the finitude argument gives a principled, non-anthropomorphic reason why code cleanliness still matters even if humans stop reading code. Where it is weakest: the essay leans on anecdote ("in my limited experience") for its central empirical claim — that AIs can't produce invariant-preserving designs — and its SaaS conclusion, while argued via Amdahl's law, conveniently flatters incumbent interests; the HOA-regulatory thought experiment is doing rhetorical work a serious counterargument (e.g., that *averaging over* real-world exposure via large-scale simulation could substitute for lived exposure) never gets to answer. The moderation coda is the most quietly original part: a community's practical definition of a valuable project — "does anyone other than the author run it?" — is the same value function the essay derives abstractly.

## Relations

- [[Coding Is Not Solved — Alex Ewerlöf]] makes the same creation-cheap/accountability-expensive split from the practice side; jerf's testing-against-reality is the mechanism behind Ewerlöf's ownership argument, and both conclude SaaS economics shift toward selling proven trust rather than code.
- [[The Cost YAGNI Was Never About]] shares the worry that cheap generation amplifies rather than removes cost traps; jerf sharpens it from options-pricing to an accounting metaphor — the LOC liability that someone will now put on the benefits side of the chart.
- [[The Flat Curve Society]] predicts SaaS roaring back once discernment becomes the bottleneck; jerf supplies the structural reason — value concentrates in the codebases with the most real-world exposure, so the tested-vs-summoned gap widens.
- [[AI Killing B2B SaaS]] is the direct opposing thesis (bespoke replaces SaaS); jerf's Amdahl's-law argument is the strongest single rebuttal this wiki holds against it.

---

*Sources: [[raw/what-value-code-in-ai-era]], [[summary/what-value-code-in-ai-era]]*
*Last updated: 2026-10-03*
