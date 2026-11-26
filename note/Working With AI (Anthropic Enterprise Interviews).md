# Working With AI (Anthropic Enterprise Interviews)

Anthropic researchers conducted 66 face-to-face interviews across 58 enterprise customers to answer how businesses actually adopt AI — and the answer is organizational, not technical. Value tracks organizational change: the lightest-touch deployments are rated lowest value, while the highest-value ones required rethinking roles, reporting lines, and who owns a workflow. Layoffs from AI are rare in the sample, but "we're not firing, we're not hiring" and cuts to offshore contractors are the quiet displacement channel.

---

## What the report argues

Three adoption strategies, each with a characteristic failure:

- **Top-down automation** is brittle. Vendor-built point solutions go stale as models improve ("It'll be obsolete in three months"); experts resist building their own replacements; surveillance use cases (daily AI throughput reports per employee, a "panopticon effect") shift power upward.
- **Bottom-up experimentation** produces dramatic individual wins — an accountant vibe-coding an 80-hour 401(k) audit into an app — but stays stuck in pockets, imposes maintenance costs nobody budgeted, and creates political friction ("Claude grenades," "I'm not reading your AI email").
- **Reorganization around AI** — broadening roles, redeploying people to bottleneck tasks, and shared infrastructure like democratized company data — is where the durable value lives.

The theoretical frame is Paul David's electric mills: technology is installed before the organization knows how to use it, and individual contributors lack the authority to rebuild the division of labor.

## Key quotes

> "We may not be firing, but we're not hiring."

The most important single sentence in the report for the labour question. Direct displacement is rare in the sample; the effect shows up in forgone hiring and, especially, in nine companies cutting offshore contractors — an early warning for the international business services industry, not for domestic knowledge workers yet.

> "If I'm open to this, I'm essentially building my own coffin."

The core incentive problem of top-down agentic automation: the workflows need the very experts they threaten, and the experts know it. The report shows this producing real resistance — reviewers stretching "two-minute" reviews back to ten-plus minutes, a conflict-checking agent shelved after staff balked at the hard cases it routed to them.

> "You want to make your work legible to your manager. But nowadays I think you want to make your work legible to the agents your manager uses."

A quiet, chilling reframe of what "working out loud" means when the reader is a monitoring pipeline. Anthropic notes the monitoring uses it heard about complied with its usage policy, but the ambivalence of the executives describing it is the interesting data.

> "The closer you are to the money, the harder it is for models to excel in that job."

A neat empirical generalization from the field: prototyping is fast, deployment isn't; production is fast, marketing isn't. Bottlenecks migrate rather than disappear — code review, then requirements gathering, then taste.

## Key themes

#concept organizational-reorganization-as-adoption #pattern bottleneck-migration #concept labour-effects #tool nontechnical-coding

## Opinion

This is the most useful kind of enterprise-AI research: qualitative, honest about its non-random sample, and structured around mechanisms rather than hype. Its boldest and least examined claim is that reorganization "benefits both employees and employers" — the case studies are self-selected Anthropic customers telling Anthropic researchers pleasant stories, and the paralegal whose "secret sauce" got absorbed into firm-wide process is a reminder that reorganization is also about extracting and socializing individual advantage. The report's own data (two-thirds reducing hiring) undercuts its optimistic framing more than it admits.

The bottleneck-migration finding connects directly to the review-fatigue literature: as coding accelerates, review and verification become the binding constraint, which is exactly what sources like [[The End of Code Review]] and [[Agentic Code Review]] argue from the engineering side. And the "build common infrastructure" prescription — democratized data, guardrailed nontechnical building — is the enterprise-scale version of what [[The Year of Internal Tools]] documents at one small company.

## Related pages

- [[The AI Productivity Paradox]] — this report's Paul David framing and bottleneck findings are the field-research counterpart to that page's macro argument; it strengthens the claim that gains stall at organizational boundaries.
- [[Running an AI-Native Engineering Org]] — Gondala's practitioner field report reaches the same conclusion (rebuild the pipeline, not the tasks) from the inside; Anthropic supplies the cross-industry evidence.
- [[Why Generalists Could Thrive in the Age of AI]] — the report's role-broadening thesis ("collapse designer, PM, engineer, and QA into a singular role") is the organizational mechanism behind that page's individual-level argument.
- [[Nicole Forsgren on AI and Developer Productivity]] — complicates it: Forsgren's survey data on AI's uneven productivity effects pairs with these interviews' finding that accelerated individuals create organizational friction, not just gains.

---
*Sources: [[raw/working-with-ai]], [[summary/working-with-ai]]*
*Last updated: 2026-11-26*
