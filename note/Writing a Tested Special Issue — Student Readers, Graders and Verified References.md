# Writing a Tested Special Issue: Student Readers, Graders and Verified References

A one-day process report for producing a tested, referenced ten-article teaching series with agents: verifiers check references before drafting, scripts recompute every quantitative claim, weak "student reader" sessions pick the structure and answer pre/post quizzes, and stronger graders confirm their complaints against the text. The thesis: arrange as many checks-against-something-outside-the-author as possible, cheaply, before publication.

---

The document improved most at the points where a claim in it was checked against something outside the author. Everything else in the report is machinery for manufacturing those moments. What makes it unusually useful is the honesty about which measurements worked and which were vanity.

## Key quotes

> The weak readers did not measure learning. They located defects.

The sharpest distinction in the piece. Model readers-as-instruments are defect finders, not pedagogy evaluators — a framing that dissolves the usual "can an LLM judge my writing?" question by splitting the job in two.

> A reader with the text open and the ability to search it will find any answer the text contains, so this number shows only that the answers are present. It cannot rank articles and should not be reported as evidence of teaching.

The post-reading quiz score (11.9–12.0 of 12) was useless; the pre-reading score (7.6) exposed that 24 of 60 questions needed no article at all. Round 2's criterion — a correct answer must require something only in the document — dropped unanswerable-without-the-article questions from 24 to 11. This is a clean, reusable test for whether any comprehension quiz tests the document or general knowledge.

> Do not read reader feedback raw. Pass it through a checker that can open the text and the sources, and require quotations.

Weak readers produce many complaints, some wrong; strong graders with text and repository access decide which are real. Neither role suffices alone — the pipeline works because the roles are deliberately different models with different jobs. Graders also went beyond their brief, opening experiment outputs to settle disputes and reporting their own defects.

> Verify references before drafting, not after. A corrected finding changes what a paragraph argues, and paragraphs are cheaper to write once.

Of 107 memory-cited references, 103 confirmed, 4 corrected, one claim dropped — but the bigger win was verifiers' answers to specific questions ("confirm it introduces widening") replacing remembered impressions with the sources' own figures. Verification was also honest about reach: unscriptable publisher sites got cited for existence only, with no figures attributed.

> For every quantitative claim, ask whether it can be recomputed from the repository in under an hour. If it can, recompute it and keep the script.

And the catch: reading the build configuration exposed an error in the brief that no reader would have caught, because it was consistent with everything else on the page. Some errors only something outside the text can find.

## Themes

#concept #pattern #tool

- **Five independent checks, each with a defined role**: verifier (web access, checks references), student reader (weak model, only the page), grader (stronger model, text + repo access, must confirm complaints), author (one voice), instruments (scripts producing numbers).
- **Before/after reading as an experimental design**: two fresh sessions per reader per condition, no shared memory — independent samples. The before condition is "the only part of the quiz that tests the quiz."
- **Everything writes files**: agents return short summaries and put full output on disk. The author's context stays small and the whole run is auditable — a thousand session files as the record.
- **Edits as exact replacements with assertions**: revision scripts of (old, new) pairs that halt if old text doesn't occur exactly once, making each round's changes reviewable and un-silent.
- **Costs are itemised**: ~430K–523K output tokens per 60-session round (~20–25 min elapsed at ten in parallel), verification agents 21–24 min each — the stronger models were the larger expense and produced the larger share of confirmed findings.
- **Known limits stated**: one model for all readers, spot-checked (not repeated) grading, figures unreadable to readers, final corrections untested. The count of confirmed defects did not reach zero.

## Opinion

This is the best documentation I've seen of agentic *document* testing as an experimental design rather than a vibe. The load-bearing insight is role separation: the same property that makes review work (weak finder, strong verifier) applied to prose. Its critique of post-reading quiz scores deserves to become standard practice for anyone using "did the model understand it?" evals — an open-book score measures retrieval, not teaching. The report also quietly demonstrates the value of doing verification *before* generation: four of five stages cost minutes, and the whole thing fit in a day precisely because checking was parallelised ahead of writing, not appended after it.

The honest caveat — model readers can't establish that humans learn — is also the boundary of the method. What it buys is defect density, at measurably low cost; what it can't buy is a judgement about whether the result is good. The author keeps that judgement, and the report is better for saying so.

## Related pages

This source strengthens [[Agentic Verification]] by carrying Osmani's "checking is cheap, build loops" argument out of code and into long-form prose, with the same conclusion: quality comes from arranging checks the author/agent can run, not from reading more carefully. It nuances [[Ways of Checking]] with a concrete five-check taxonomy and the pre/post quiz design as a way of testing the checker itself. It complicates [[Four Writing Humans (No Wait, Five)]] — that piece maps writing roles among humans; here the same role separation is done with models of deliberately different strengths. And it offers a counterpoint to [[Make Better Documents]]: where that page works on the writing act, this one works on the measurement of the written thing.

---
*Sources: [[raw/project-report-writing-a-tested-special-issue-student-readers-graders-and-verified-references-a-technical-deep-dive]], [[summary/project-report-writing-a-tested-special-issue-student-readers-graders-and-verified-references-a-technical-deep-dive]]*
*Last updated: 2026-11-20*
