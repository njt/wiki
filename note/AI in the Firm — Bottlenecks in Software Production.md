# AI in the Firm: Bottlenecks in Software Production (Chen & Stratton)

The most rigorous empirical test yet of whether AI coding gains actually reach the firm: 300M work events across 718 firms show agents raise individual coding output ~30%, but software output and employment barely move — because code review becomes the bottleneck, absorbing the extra volume as slower reviews, more changes requested, and labour reallocated toward verification.

---

This is a Harvard job market paper (Fiona Chen & James Stratton) built on Jellyfish's engineering analytics platform: ~300 million work events — GitHub commits and pull requests, Jira issues and epics, Google Calendar, HR data — covering 726K workers across 718 firms from January 2021 to March 2026. Identification is a staggered difference-in-differences on firm-level adoption timing, exploiting quasi-random variation in procurement process length (legal review, security approval, piloting — one firm took months, another a day).

The headline numbers, cleanly separated by tool generation:

- **Assistants** (Copilot, Cursor): +12% lines of code, +9% commits, +5% PRs — only commits significant.
- **Agents** (Claude Code, Cursor Agent, Devin): +30% LOC, +20% commits, +23% PRs — all significant.
- **Output** (Jira issues and epics resolved): small, insignificant. Agents rule out gains above 12%, against a 30% productivity gain.
- **Employment**: precise zero for agents (declines greater than 2.9% ruled out overall).

The gap between 30% and ~0% is the paper's real contribution, and it locates the lost productivity precisely: in code review. Review time rises 49%, the share of PRs with changes requested nearly doubles, comments per PR rise 35%, and the share of workers doing review rises 14%. Engineers spend only 30–40% of their time writing code; the pipeline downstream absorbs everything the agents generate. Crucially, the bottleneck persists even after firms adopt AI code review tools — human reviewers stay central.

---

## Key quotes

> "For AI agents, this incomplete pass-through reflects a bottleneck from code review: review times increase, a larger share of code updates require revisions, and reviews involve more comments."

The abstract in one sentence. The framing matters: this is a *production* problem, not a capability problem. Agents work; the organisation doesn't.

> "A firm's staffing mix therefore shifts toward review — and never toward coding — as AI adoption raises coding productivity or the bug rate."

Corollary 2 of their model is the cleanest result in the paper. Both channels — more code volume and worse code quality per unit — push labour toward review. The "oncoming white collar bloodbath" narrative runs into this: the observed firm-level response is reallocation, not cuts.

> "Although firms increasingly use AI to assist with code review, human reviewers continue to play a central role in the review process."

The result that complicates the automation-of-review optimists: the AI-review tools exist, firms adopt them, and the bottleneck remains. Review capacity itself doesn't scale with AI yet.

## Key themes

#concept bottleneck-tasks #concept incomplete-pass-through #tool jellyfish-engineering-analytics #pattern review-as-constraint

## Analysis

This paper is the empirical anchor the wiki's review-bottleneck literature has been missing. Much of what's collected here — [[The End of Code Review]], [[Agentic Code Review]], the whole AI Code Review topic — argues from practitioner observation and design intent. Chen and Stratton bring difference-in-differences over 718 firms and land on the same conclusion from the other direction: the bottleneck is real, measurable, and *worsening* under agents (+49% review time), not merely anecdotal. It strengthens [[Writing Code vs. Shipping Code]] — Demirer et al.'s open-source finding that commit gains attenuate at release level — with the firm-internal version of the same attenuation curve, and it adds the missing causal mechanism that paper could only hypothesise: review friction. It also supplies the hardest numbers behind [[Five Studies That Are Changing How I Think About AI in Software Engineering]]'s synthesis that downstream is breaking while upstream accelerates, and it complicates [[Nicole Forsgren on AI and Developer Productivity]]'s measurement agenda by showing which firm-level metrics actually move (activity up, output flat).

Two things deserve scepticism. First, the outputs measured are Jira issue resolution counts — a proxy that can be gamed or mismeasured, which the authors partially address via ML-predicted issue length but not fully. Second, the "bug rate" channel is inferred from review friction (changes requested, comments) rather than observed defects; more review comments could equally mean more noise or more scrutiny. The model's two channels are identified by assumption as much as by data. Third, the data partner (Jellyfish) sells exactly this kind of analytics — the paper is admirably transparent about review boundaries, but the sample skews to firms that buy engineering analytics, i.e., larger and more instrumented firms.

The deepest implication is for the employment debate. Against Amodei's "white collar bloodbath", the firm-level data shows a precise zero on employment with a large productivity gain — the labour is absorbed, largely by moving toward review and verification. That matches the model's starkest prediction: staffing mix shifts toward review *and never toward coding*. The white-collar-bloodbath question becomes a question about whether review itself can be automated faster than code generation.

---

*Sources: [[raw/jellyfish-pdf]], [[summary/jellyfish-pdf]]*
*Last updated: 2026-10-08*
