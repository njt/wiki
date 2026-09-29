# How to Prepare for AI-Driven Code Modernization Projects

Anthropic's forward-deployed engineers turn field experience with enterprise code modernization into a six-step preparation playbook — target, certificate, promotion policy, staging, workflow, pilot — whose core claim is that once agents write the changes, the bottleneck migrates from producing code to mobilizing the organization around it. The durable artifacts are not the modernized code but the machine-checkable certificate, the pre-agreed tiered review policy, and the evidence trail.

---

## What it argues

Legacy modernization that used to be a multi-year all-hands can now finish in weeks or months — but the organizational work on either side is unchanged. Change management, review, and approval exist because regulators and auditors demand them, and they were designed for human-authored diffs. Agent acceleration breaks that assumption, so the preparation work is to re-specify what evidence a change must carry and who signs off on what.

The six steps:

1. **Target** — decide which of three modernizations you are doing: *transform* (swap the stack, hold behavior constant — what production-adjacent people want, to contain risk), *reimagine* (pay down tech debt plus new requirements — what the engineers who live in the code want), or *uplift* (modernize in place on a live codebase). Leaving this unresolved guarantees it resurfaces as an argument over whether a change is "correct." Discovery — Claude's *assess*, *map*, and *extract-rules* commands mining business rules with source citations — plus interviews and internal docs informs the choice. Cost reduction is rarely the real driver; **risk of not modernizing** is the counterweight that settles stakeholder arguments.
2. **Certificate** — the set of machine-checkable conditions every change must meet (behavioral equivalence via replay harnesses and prod-parallel setups, tests, security scans, performance). Each condition must be checkable *without a human in the loop*, so the agent can iterate until it passes or self-flags. Write it with the people who will review and promote changes; the test is whether they'd merge on the certificate's evidence alone. Where legacy systems have thin coverage and no telemetry, part of the step is using Claude to *build the missing evidence*.
3. **Promotion policy** — a tiered review path written down and agreed *in advance*, because agents produce changes faster than any team can review diff-by-diff. The key inversion: SME hours are front-loaded (they tune the certificate and bless early samples), which earns lighter review later — the reverse of the traditional pattern where review happens at the end. In regulated environments, individual approvers hesitate because they carry the risk; the fix is a directive from the top and pre-agreement so responsibility for a production bug is shared, not pinned on the last approver.
4. **Stage** — the run depends on teams outside the modernization (platform, QA, security, compliance), each with its own backlog, so open those conversations during steps 1–3.
5. **Build the workflow** — start from the code-modernization plugin, put everything the workflow needs (target, certificate, policy, codebase, docs, tooling) on the filesystem or over MCP, and have SMEs review Claude's extracted rules and skills *before* anything downstream relies on them. When issues surface, **modify the workflow, not each change** — the goal is confidence that scaled output meets the certificate almost everywhere.
6. **Pilot, then scale** — run end-to-end on a small slice including real review and landing, measure tokens and extrapolate, treat what the pilot couldn't see as unknowns. Uplift runs partition from the leaves inward, freeze one partition at a time, and gate CI/CD so new commits can't undo a modernized partition.

## Key quotes

> "Those processes are what make critical systems trustworthy, and they were built on the assumption that a human wrote each change and a human would review each diff. Once agents accelerate writing the changes, the bottleneck shifts from producing changes to mobilizing the organization around them."

The thesis in one sentence, and a more sophisticated framing than the usual "AI is fast, process is slow" complaint: the process isn't an obstacle to remove, it's the thing that must be re-specified for agent-scale throughput.

> "Each condition should be checkable without a human in the loop, so the agentic workflow can iterate on a change until it meets the certificate or flag it for human review if it can't."

The certificate is exactly a verification harness expressed as an acceptance contract — the same move as spec-driven development, but with the twist that the spec is written to be *agent-iterable*, not just human-readable.

> "You should modify the workflow, not each change, when issues surface."

The single most transferable line in the piece. Fixing individual changes during a pilot is a category error — the unit of correction is the loop, not the diff. It's the software-factory discipline stated as an operating rule.

> "It is best to have the directive for the promotion policy come from the top of the organization. It is also better to agree on it beforehand so responsibility for a bug that reaches production is shared, not pinned on whoever approved the change."

Bluntly political, and refreshingly so: the real blocker in regulated modernization isn't technical capability but liability allocation. The article names it and prescribes an org-design fix rather than pretending better tests dissolve it.

> "Move compute-heavy verification signals behind cheaper gates so they only run once easier checks have passed. ... Keep more intelligent models for hard transformations and the adversarial reviews that verify correctness."

Cost engineering as model routing by verification tier — the certificate doesn't just define correctness, it defines *which model is allowed to attempt which work*.

## Key themes

#concept #pattern #tool

- **The certificate** — a machine-checkable definition of "done" that doubles as the agent's iteration target and the human reviewers' evidence base.
- **Front-loaded review** — SME effort moves from end-of-pipeline diff review to start-of-pipeline certificate and sample review, buying lighter review at scale.
- **Responsibility design** — pre-agreed promotion policies and top-down directives as the answer to approver liability paralysis in regulated environments.
- **Pilot-as-cost-model** — measure token usage on a slice, extrapolate, route models by verification tier, analyze retry economics.

## Critical take

This is vendor advice — Anthropic selling its forward-deployment practice and its code-modernization plugin — so the six steps should be read as a product-shaped methodology. But it's vendor advice of the useful kind: it names the actual failure mode (organizational mobilization, not model capability), and its two sharpest ideas — the machine-checkable certificate and front-loaded SME review — are tool-agnostic and verifiable against the reader's own change-management reality.

The weak spot is the certificate's circularity: in a *transform* modernization, the evidence of behavioral equivalence is largely built by the same agent whose output it judges (Claude writes the replay harness, the tests, the prod-parallel setup). The article gestures at this — SMEs review extracted rules "before anything downstream relies on it" — but never confronts how much trust in the certificate rests on trusting Claude's evidence-generation in the first place. The honest version of this guide would spend a section on who audits the auditor's harness. It also understates the politics: "building internal consensus" gets one paragraph, while anyone who has run a regulated modernization knows that is ninety percent of the project.

Still, as a complement to practitioner essays on brownfield agent work, this is the enterprise/regulatory lens most of them skip — and the "modify the workflow, not each change" rule deserves to outlive the product it's attached to.

## Related pages

This source is the missing regulated-enterprise, org-design layer for [[Brownfield Agentic Engineering (Osmani)]] — Osmani's zone maps and trust-building for legacy codebases are the technical half of what this article's certificate and promotion policy formalize for critical systems.

It directly extends [[The Archaeologist's Copilot]]: Malykhin's discovery-first instincts (Tourist vs. Archaeologist prompts, containment-first) match the *assess/map/extract-rules* step here, but Anthropic adds what a solo practitioner's report lacks — the change-management and liability machinery around the discovery.

It sharpens [[Harness Engineering in Practice]]: Frisinger's fail-closed gates and evidence manifests are single-repo versions of the certificate, while this article shows what the same idea becomes when the review path must satisfy regulators and the policy must be agreed before the first change lands.

And it complicates [[A Practical Guide to Brownfield AI Development]] by insisting the hard part is neither the code nor the agent but the sign-off structure — a dimension that guide's practitioner workflow largely leaves implicit.

---
*Sources: [[raw/how-to-prepare-for-ai-driven-code-modernization-projects]], [[summary/how-to-prepare-for-ai-driven-code-modernization-projects]]*
*Last updated: 2026-09-29*
