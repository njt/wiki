# How Much Code Do Developers Really Let Agents Write (JetBrains 2026)

JetBrains' Developer Ecosystem Survey 2026 asked over 15,000 professional developers what percentage of last month's work code was fully agent-generated, AI-assisted, or fully manual. The result is the first large, globally representative measurement of actual code provenance rather than tool adoption: ~47% of code is fully agent-written on average, but only ~22% of developers are heavily agentic, and ~23% still write three-quarters of their code by hand. The claim that "100% of my code is written by agents" describes a vocal minority, not the profession.

---

## The argument in one paragraph

The average professional developer now sources roughly half of their production code from agents (~47% fully agent-generated, ~38% AI-assisted, ~27% manual), but the distribution is not a bell curve around that mean — it is a profession splitting into three distinct segments (agentic ~31%, AI-assisted ~47%, manual ~23%), with heavy agentic adoption concentrated among seniors, Codex users, Go/JS/TS developers, and East Asia, and lagging among juniors, C/C++ developers, and Europe. If this claim is wrong, it is wrong in a specific way: self-reported bucketed percentages from a survey run by a company that sells developer tools may not track actual code provenance, and the bucket-midpoint arithmetic (which JetBrains admits can sum to over 100%) is soft. But the shape of the finding — rapid collapse of pure manual coding, slow growth of full agentic workflows — is the kind of thing that other data sources can and will corroborate or refute.

## Key quotes

> ~47% of their code is fully written by agents.

The headline number, and the one that will be misquoted most. It is an average across a bimodal population — the median developer is not writing half their code with agents; they are either well above or well below.

> the group that relies almost entirely on coding agents (over 80% agent-generated code) remains a minority of around 22%

This is the sentence that deflates the discourse. Two years into the agentic era, four out of five developers still write at least a fifth of their code themselves — and a quarter write most of it manually.

> Interestingly, **senior developers are among the first to hand coding over to agents**.

This inverts the common narrative that juniors adopt AI fastest because seniors have the most to protect. Seniors appear to trust agents with generation precisely because they can review the output — a claim the survey supports only indirectly, but one with real consequences for how junior engineers learn the craft.

> while Claude Code is increasingly becoming the mainstream AI coding tool (already the de facto standard, with 39% adoption at work), its audience no longer consists predominantly of advanced users of agents

A quietly important market observation: tool adoption and workflow depth are decoupling. Claude Code winning the market share war does not mean its users are the most agentic — Codex users (42% heavy-agentic vs 32%) out-agentic them, plausibly because higher quotas attract heavier use.

> About twice as many developers there (32%–35%) generate the vast majority of their code (over 80%) with agents, compared with around 16% of developers in Europe and the UK.

The regional gap is the most under-discussed finding here. A two-fold difference in heavy agentic adoption between East Asia and Europe is not a cultural curiosity; it is a leading indicator of where workflow innovation will happen first.

## Critical analysis

The non-obvious value of this source is that it measures *code provenance*, not tool usage. Nearly every other survey in this space asks "do you use AI tools?" — a question that saturates at yes and tells you nothing. Asking what fraction of last month's code was agent-written, AI-assisted, or manual is a much harder question to answer and a much more informative one. The three-segment clustering (agentic, AI-assisted, manual) is the right cut, and the finding that the middle segment is the largest — developers who use agents heavily but refuse to go full-agentic — matches what practitioners report anecdotally: the last 20% of hand-written code is where judgment lives.

The weaknesses are real, though, and JetBrains is honest about them. Self-reported bucketed percentages are noisy: respondents estimating what fraction of a month's code fell into each category is barely a measurement, and the methodology notes concede that bucket midpoints can make the three categories sum above 100%. The midpoint arithmetic (10.5%, 30.5%, 50.5%...) manufactures precision the data cannot support. There is also a selection question the report waves at but does not resolve: a survey weighted by "familiarity with JetBrains products" is still a survey of people willing to take a long developer survey, and heavy agentic coders may be overrepresented among the online-survey-taking population.

What the source leaves out is the consequence layer. It measures how much code agents write but says nothing about what happens to that code — defect rates, review burden, rework, or whether the 22% who are fully agentic ship faster or better. It also cannot distinguish "developers choose their workflow" from "the codebase chooses it": the C/C++ gap is almost certainly about tool fitness, not developer conservatism, and the seniority gap may be about codebase ownership rather than trust. And the claim that seniors hand coding over first deserves its own study — if the people who review best are the ones who delegate most, that is a strong argument that review skill, not generation skill, is the bottleneck of the agentic era.

## Related

- [[Laura Tacho — Data vs Hype]] — Tacho's 121,000-developer dataset established that 92.6% of developers use AI tools monthly; this survey strengthens her anti-hype stance by showing that near-universal tool adoption coexists with only ~22% of developers actually going agentic, exactly the adoption-vs-transformation gap she argues for.
- [[You Shall Not Pass — Where Developers Draw the Line on AI Autonomy]] — That page maps where developers draw autonomy boundaries by task; this source complicates it by showing the boundary is not one line but three population segments, with the "AI-assisted" middle refusing full delegation even after heavy agent use.
- [[Five Levels from Spicy Autocomplete to the Dark Software Factory]] — The five-level maturity model gets its first population-level distribution here: the survey's agentic/AI-assisted/manual segmentation reads as an empirical snapshot of where developers actually sit on that ladder, with the dark factory levels still a minority position.
- [[Eight Myths of AI in Software Engineering]] — The Microsoft researchers warn that adoption metrics mislead; this source is a partial counterexample — a survey that asks about code provenance rather than tool usage — but its self-reported buckets also illustrate exactly the measurement fragility that page describes.

---
*Sources: [[raw/how-much-code-do-developers-really-let-agents-write]], [[summary/how-much-code-do-developers-really-let-agents-write]]*
