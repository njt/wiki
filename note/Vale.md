# Vale

Vale is a prose linter built on the premise that documentation, like code, has parseable structure — so its checks can be scoped to headings, lists, and table cells rather than applied indiscriminately. Its homepage leads with the tagline "Your document has structure. Your rules should too," and reports support for twelve markup formats, from Markdown and AsciiDoc to DITA, Org, and QDoc. This page is a thin slice of the product (the Scopes feature only), but the framing is the substance: prose is lintable the way code is.

---

## Key quotes

> "Your document has structure. Your rules should too."

The whole pitch in two sentences. It asserts that prose deserves the same treatment code gets from linters — and that genericity is the enemy. A rule that fires on code blocks or link URLs is noise; a rule that fires only inside table cells is signal. This is scope-as-precision, borrowed wholesale from compiler thinking.

> "Vale parses the markup first, so code blocks and link URLs stay out of the way."

The mechanism behind the tagline: parse the document into an AST, then apply rules against nodes, not raw text. The claim quietly implies Vale knows the difference between prose and content-that-looks-like-prose, which is exactly the failure mode of regex-based doc checks.

> "12 markup formats" (Markdown, AsciiDoc, reStructuredText, MDX, MyST, Quarto, Typst, HTML, XML, DITA, Org, QDoc)

The breadth is the deployment argument: one rule set travels across a team's entire documentation stack. Rules-as-infrastructure only compounds if they aren't per-format bespoke.

## Themes

#tool #concept #pattern

## Analysis

This homepage is nearly empty as a document — a tagline, a feature name, and a format list — but it lands on one of the load-bearing ideas of the guardrails topic: deterministic checks work best when they are *scoped*, because precision is what separates a linter from an alarm system. The Scopes feature is really an argument about false positives. Blanket rules punish the innocent (code blocks, URLs); scoped rules earn the right to block.

It's worth being honest about the limits of the source: this is marketing copy, not a field report. There are no numbers, no benchmarks, no war stories. What redeems it in the wiki is the clarity of the framing and the twelve-format claim, which make Vale a concrete anchor for the argument that prose quality is machine-checkable.

The most interesting unexplored thread is agentic. If a coding agent writes documentation — changelogs, ADRs, design docs — a scoped prose linter is exactly the kind of cheap, unfakeable verification that belongs in the loop: deterministic, fast, and immune to the model grading its own homework. Vale's structure-first design makes it unusually well suited to that role, because rules that ignore code blocks won't punish an agent for writing a code-heavy doc.

## Related pages

This source strengthens [[Deterministic When Possible, Probabilistic When Necessary, Human When Cheaper]], which ranks deterministic tools first and agents only at the gaps — Vale is a textbook example of a deterministic tool that should run before any LLM proofreads anything. It nuances [[Code Standards That Survive the Next Agent]]'s CI-enforcement argument by extending "what every change must prove" from code into prose. And it connects to [[Writing Style Guides for Better UIs]]: Langworth feeds a style guide to an agent to enforce it; Vale is the deterministic half of that same loop, catching violations without a model call.

---
*Sources: [[raw/vale-sh]], [[summary/vale-sh]]*
*Last updated: 2026-10-11*
