# Design Language in Three Days (curiosity.ai)

The curiosity.ai team's account of rebuilding their entire brand — 67 pages of website, blog chrome, docs, and slide decks — with coding agents: from a Bauhaus-poster experiment on a Friday afternoon, through twenty-two full-site design directions judged in a replayable pairwise showdown, to a `BRAND.md` spec written for an agent that "has no eyes," and a three-day, ~200-commit rebuild. It is one of the most concrete field reports yet on how taste, specification, and verification actually divide between human and agent.

---

## What happened

A NYT Bauhaus article prompted the question "what would curiosity.ai look like as a Bauhaus poster?" The next afternoon, the whole site was rebuilt in that style — fast because the site is a small static generator with shared modules. Rather than stop, they generated twenty-one more whole-site directions (Swiss minimalism, 1984 desktop dither, a departure board, a pixel-rasterized site), each a full copy with its own stylesheet and diagram code, all carrying identical 67 pages and identical words so any two could be compared one click apart. A knockout tournament (47 matches, ~12 minutes) picked **Yarn Mosaic**, a pixel-cat aesthetic. In parallel, design work produced `BRAND.md`; the final site merges the mosaic's structure with the brand's monochrome-plus-one-blue system, and a second tournament inside the brand picked a blend of section winners.

## Key quotes

> "Success criterion for any asset: could another agent reproduce it from these rules without seeing it? If a choice cannot be written as a rule or a number, do not make it."

`BRAND.md`'s own first rule. This is specification-driven development applied to visual identity — taste is not encoded in the agent, it is encoded in the brief, and anything the brief can't express as a rule is explicitly reserved for human review. It's the strongest single formulation of the "specs are the product" thesis I've seen applied outside code.

> "'Modern and clean' produces something average. 'The escaped square is 20 units on a 24 unit grid, one cell out on the diagonal' produces the mark."

Numbers versus adjectives as the dividing line between what an agent can execute and what stays mushy. Underneath it is a real epistemology of delegation: if the rule can't be applied without seeing the result, "it is taste, and taste belongs to a person reviewing the page."

> "The responsive checker reported a clean pass on every page for its whole early life without inspecting one. … A checker that cannot tell 'looked and found nothing' from 'did not look' is worse than none."

The most useful lesson in the piece, by the authors' own account, and a genuine contribution to guardrail design: their checker had a swallowed error and a wrong call signature, so it passed everything vacuously. The fix — printing how many page widths it actually looked at, and failing the build when a diagram label drops under 9px — is "build the checks, then check the checks" made concrete. 43 sub-9px labels and 16 comparison pages had passed every check while being illegible on phones.

> "A model under pressure to fill a slot will fill it with something plausible. … plausible terms that look finished are worse than an empty page."

Content frozen while the look moved: the brief lists the publishable numbers, and a section without an approved proof point stays empty and gets flagged. The DPA page shows structure with no text until legal wording arrives. This is a specific, operational answer to the LLM plausible-fill problem that most "AI writes our site" stories never confront.

> "Every change ended with the page rendered in a browser and looked at, by the agent and then by us. … Overflow checks pass on pages that are ugly. Only looking catches ugly."

The whole pipeline still bottoms out in a human (and an agent) looking at the rendered page. Every PR was merged by a person who looked at it on the rendered page — the code was written by Claude Code, but the merge was an act of taste.

## Key themes

**#pattern — Taste as specification.** The brief is the product: palettes with hex values, a Signal budget per section, rulings with reasons, and a record of what was tried and rejected (scroll snapping, a pixel typeface, flamingo pink) so nobody tries them again.

**#pattern — Cheap variation + pairwise judgment.** Make a variation cost minutes (a palette on an arrow key, a direction as a site copy, three page layouts side by side, keep one), then decide pairwise with a replayable 14-character vote code. This converts taste from an argument into something "you can count."

**#concept — AI-first as architecture.** Everything published is Markdown an agent can read and write; every repo carries the instructions; the blog's chrome is synced from the website's own stylesheets rather than re-created. "When the brand changes, one file changes, and the next session reads the new one."

**#concept — The vacuous check.** A guardrail that can't distinguish "looked and found nothing" from "did not look" actively misleads. Their fix is making the checker report its own coverage.

## Critical analysis

The honest part of this piece is that the agent produced plausible-looking failures, not just plausible-looking wins: 43 illegible labels, a checker that lied for its whole early life, `filter: url()` pointing at nothing making elements vanish silently. The authors treat this as the headline lesson rather than burying it, which puts the piece well above the average "we vibe-coded our brand" essay.

Less examined: twenty-two full-site directions were generated in a few days, and twenty-one were deleted "with their tooling." That is a deliberately wasted majority, and the piece is upfront that the point of cheap experiments is affording the strange ones — but the economics only work because the site is a 67-page static generator with shared modules and fixed copy. The same method on a large application with dynamic content and per-page logic would not reduce to a palette switch and a stylesheet copy. The transferable part is the protocol (fixed content, cheap variants, pairwise replayable judgment, numbered briefs), not the speed.

There's also a quiet tension between lesson 5 ("specs should be numbers") and the brand's own playfulness: Sudo the walking pixel cat is "a deliberate exception, and it is written down as one." The system works because the spec has an escape hatch and the escape hatch is itself specified. That's a more sophisticated picture of specification than the usual spec-driven absolutism — a good spec knows which exceptions it is granting.

## Related pages

- [[Turning Your AI Into a World-Class Designer (Chimala)]] — the closest sibling: Chimala's design-critic subagent loop reaches the same conclusion (human judgment as the scarce input) via a different mechanism; this piece strengthens it with a full production case, including judging real pages over mockups.
- [[Undertone — Taste Elicitation as a Creative Brief]] — both treat taste as an elicitation-and-comparison problem rather than a generation problem; this source's pairwise showdown with a replayable vote code is what Undertone's guided comparisons look like when the thing being judged is your own production site.
- [[Writing Style Guides for Better UIs]] — Langworth's "feed a style guide to an agent" pattern scaled up to a whole brand system; this piece nuances it by showing the guide (`BRAND.md`) must be written for an agent with no eyes and must log its own rejected options.
- [[Specifications as the Product]] — the compiled case for specs as the durable artifact; this source is a vivid instance where the spec is the entire brand and "the next session reads the new one" is literally true.

---
*Sources: [[raw/2026-09-design-language-in-three-days]], [[summary/2026-09-design-language-in-three-days]]*
*Last updated: 2026-10-10*
