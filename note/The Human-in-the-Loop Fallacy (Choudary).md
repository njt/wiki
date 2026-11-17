# The Human-in-the-Loop Fallacy (Choudary)

Sangeet Paul Choudary (HBR, Nov 2026) argues that "human-in-the-loop" — the reassuring default of enterprise AI adoption — organizes companies around the shrinking frontier of what machines can't yet do, creating human roles that are complacent, skill-eroding, and self-liquidating. His alternative: use AI's removal of execution constraints to *rebundle* the organization around newly available decisions, not around the residue of old work.

---

## The argument

The human-in-the-loop blueprint reads the transformation backwards: start from today's workflow, ask which tasks the machine absorbs, assign the leftovers to people. "People appear nowhere as a source of new productive capacity." The symptoms are already visible — Fiverr's fastest-growing writing request is "humanize AI content" (+641% in six months); correcting machine output is ~a quarter of European translation revenue; xAI dismissed ~500 generalist "AI tutors" in September 2025 while expanding specialist tiers tenfold.

Two compounding problems make gap-filling roles uniquely bad:

1. **The irony of automation.** Bainbridge's four-decade-old finding: better automation makes worse human safety layers — complacency, atrophy, inattention.
2. **Self-liquidation.** AI's failures sound as confident as its successes, so coherence-graders miss correctness errors. And every exception a human handles becomes training data: "each exception a person handles teaches the system to handle that exception; the better the data, the smaller the scope of human validation needed."

The positive program is "rebundling": when a general-purpose technology removes constraints, it changes *where the meaningful choices live*, and organizations should rebuild around the people making those choices. The Mandalorian's LED volumes didn't just improve VFX — they moved post-production decisions onto the set and created virtual production supervisor roles. BBC Verify moved journalists from reporting claims to establishing provenance. IKEA retrained 8,500 call-centre staff into remote interior design, generating €1.3B/year — expansion, not cost-cutting.

## Key quotes

> "This supposedly reassuring phrase often amounts to arranging a company's most valuable people around the very tasks that a machine is about to take from them."

The sharpest one-sentence summary of the gap-filling fallacy — the human is positioned adjacent to the automation frontier precisely where the machine is heading.

> "Its failures sound as confident, fluent, and well-structured as its successes. Reviewers who merely grade coherence fail to grade correctness."

This is why "human as validator" is worse than a weak role: it's a role whose skill requirement is illegible, so it selects for fluency-grading over truth-grading.

> "A reorganization that adds new titles but fails to account for these three shifts [firm boundaries, unit of organization, center of gravity] is merely a case of task reallocation."

The three-shift test is the article's most operationally useful artifact — a checklist for telling transformation from rebranding.

> "Turning their own workforce into trainers of someone else's advantage."

The economics coda: firms that don't internalize the learning cycle donate their exceptions, corrections, and proprietary knowledge to the tool provider.

## Themes

#concept #pattern #ai-product-and-business

## Opinionated take

Choudary is right about the diagnosis and right that gap-filling roles have "the shortest half-life" — the xAI tutor layoffs are the empirical receipt. The essay's blind spot is that the rebundling examples are mostly retrofits told as success stories (IKEA's €1.3B is a retailer's expansion narrative; Deere's is a decade-long acquisition-and-reorg arc), and the prescriptive steps ("map fallen constraints") are easier said than done when the constraint map itself shifts every few months. There's also an unstated tension with his own governance advice: he demolishes bolt-on human validation but then endorses Morgan Stanley's golden-question regression suite — which works precisely because it removes the *human* from the loop, substituting automated evaluation. The article is therefore less an argument against verification than an argument that human validation is a transitional, self-consuming stage; the durable roles are the ones that own new decisions, not those that grade old outputs.

For software teams, the translation is direct: the "reviewer of agent output" role is exactly the gap-filling job Choudary predicts will evaporate — which is an argument for building verification that doesn't depend on a human reading every diff.

## Relations

- [[Human-in-the-Loop is Tired]] — Summers diagnoses the psychological cost of the same role Choudary attacks (supervision fatigue, broken reward function); this source supplies the organizational-economic version of her individual-experience argument, plus the reason the fatigue never ends: the role is designed to shrink.
- [[Oversight Degrades the Overseer]] — the arXiv position paper on supervisory skill erosion is the mechanistic elaboration of Choudary's Bainbridge point; this essay complicates it by adding the *incentive* layer — oversight isn't just degraded, it's actively self-liquidating via training data.
- [[The AI Productivity Paradox]] — Cagan's acceleration-without-outcomes argument is the same backwards-reading failure at the product-team level: using AI to speed up the old project model instead of changing the model. Both sources are anti-efficiency-case.
- [[Organizational Intelligence Systems]] — Choudary's "internalize the learning cycle" prescription (John Deere) is the strategic rationale for that source's keep-human-review-nonnegotiable architecture; this essay nuances it by warning that nonnegotiable human review must not be residual gap-filling.

---
*Sources: [[raw/the-human-in-the-loop-fallacy]], [[summary/the-human-in-the-loop-fallacy]]*
*Last updated: 2026-11-17*
