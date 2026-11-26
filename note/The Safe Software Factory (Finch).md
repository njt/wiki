# The Safe Software Factory (Finch)

Andrew Finch's talk at ai.engineer CODE 2026 lays out Sonar's framework for making the "software factory" safe: since agents automated the coding bottleneck, the problem is confidence, and the answer is defense in depth built from four verification loops — PR checks, post-deploy verification, maintenance, and knowledge — with humans relocated to the highest-judgment positions rather than removed.

---

## The argument

Finch (staff engineer at Sonar, working on Guitar, its AI code review product) opens with the now-standard software-factory frame: the SDLC loop — inputs, planning, build, PR, review, CI, merge, deploy, feedback — where "automation has followed every bottleneck." Coding was the old bottleneck; agents automated it; the bottleneck moved to review, which is why AI code review arrived, and now automation is spreading to planning, monitoring, rollout control, and incident triage.

His CEO's line frames the whole talk: "Code is abundant, confidence is not." The rest is a taxonomy of where to buy confidence — four loops.

### Loop 1: PR checks

The familiar loop, widened. Beyond tests and AI code review, Finch enumerates what a PR gate could check: static analysis, **functional validation** (does the PR actually do what the ticket asks), coding-convention rules, accumulated knowledge from past reviews and outages, **cross-repo checks** (will this break something elsewhere), **cross-PR analysis** (conflicts with other open PRs), security review, end-to-end browser testing, and **risk assessment** — computing blast radius and grading low/medium/high, which "can actually help us determine if this is a PR that a human even needs to look at at all."

The key move is evidence-based merge gates: gate not on "a test ran" but on recorded findings, quality-gate results, vulnerabilities, and risk level, checked against team-defined mergeability criteria. Teams that do this well can then delegate trust upward — auto-approve, even automerge, for low-risk changes — freeing human review bandwidth for high-risk work.

### Loop 2: post-deploy verification

"Very much in its early stages": agents reading production logs, traces, and metrics to check that deployed code behaves as expected; agentic rollout (widen, roll back, flip feature flags); AI outage monitoring and root-causing.

### Loop 3: maintenance

A parallel loop scanning the codebase on a schedule, running the same checks as the PR stage and opening remediation PRs. Its unique value: catching what a single PR can't show — rules or vulnerabilities published *after* merge, and **slow drift**, the gradual accumulation of duplication and complexity invisible at PR granularity.

### Loop 4: knowledge

Curated company knowledge — documentation, specs, business decisions, lessons from past reviews and outages — fed into all the other loops. Coding agents with this context "do a better job the first time, fewer issues," which shrinks review-loop traffic upstream.

### The human role: dim, not dark

Can we run a dark factory? "Probably not. Not right now." Humans intervene where expertise matters: auditing risky code, verifying evidence, handling exceptions (rollbacks, incidents), reviewing architecture — "Agents are not very good at this yet, I would argue" — supplying the why (product intent, trade-offs, business decisions), and engineering the factory itself.

## Key quotes

> "Code is abundant, confidence is not."

Sonar's CEO Tar's line, which Finch borrows as his thesis. It compresses the whole 2026 bottleneck argument into six words: generation is solved, assurance is not. Every loop in the talk is a confidence-purchasing mechanism.

> "The factory runs dim, not dark."

A useful middle position against both hype poles. Finch doesn't claim lights-out autonomy is near, but he also doesn't pretend humans review everything — the dim factory is one where humans are deliberately placed at exception handling, architecture, and evidence verification.

> "We could compute the blast radius and categorize this into low, medium or high risk. And that can actually help us determine if this is a PR that a human even needs to look at at all."

Risk grading as an attention-routing mechanism, not just a check. This is the quiet radical claim in the talk: the merge gate can decide *not only* whether code is good but whether humans are worth spending on it.

## Key themes

- #concept — the four-loop safety architecture (PR checks, post-deploy, maintenance, knowledge)
- #pattern — evidence-based merge gates and risk-graded human attention routing
- #tool — Guitar, Sonar's AI code review product
- #person — Andrew Finch

## Analysis

The talk's strength is its taxonomy. "Add more loops" sounds vague, but Finch makes it concrete: the maintenance loop's insight about slow drift is genuinely underappreciated — PR-granular review structurally cannot see codebase-level decay, and a scheduled scanning loop can. The knowledge loop is also the right emphasis, and it echoes a pattern visible across this wiki: the teams getting the most from agents are the ones feeding them curated internal knowledge, not better prompts.

The weak point is the provenance. This is a product-company talk (Sonar sells code review and quality gates), and the enumerated PR checks — conventions, cross-repo analysis, risk assessment — map suspiciously neatly onto what Guitar could sell. The talk cites no metrics: no revert rates, no false-positive rates from the AI reviewer, no data on how often risk grading misroutes. "Teams who do this well" do what, exactly? The framework is plausible; the evidence for it is entirely rhetorical. Compare [[Building an AI-Native Engineering Org (Gondala)]], which makes adjacent claims but brings a 5x/<0.5%-revert data point.

Also underexamined: who audits the auditor. If auto-approve depends on the AI reviewer's risk grading, an AI review failure at the grading stage silently removes the human from the loop for exactly the PRs that need them most. Finch's evidence-based merge gate helps (recorded findings can be spot-audited), but the talk doesn't confront the failure mode where confidence is manufactured rather than earned.

Still, as a framework for organizing verification investment, it's one of the cleaner articulations of the position this wiki's [[Guardrails and Feedback Loops]] material keeps circling: quality is not a stage, it's a set of loops at different time constants, and the human's job is to sit at the loops machines can't close.

## Related pages

- [[Human Judgment Doesnt Leave the Software Factory — It Relocates]] — Finch's "dim, not dark" is the same thesis from the practitioner side: judgment doesn't vanish, it moves to auditing, architecture, and supplying the why. This talk gives that essay's argument a concrete four-loop structure to relocate into.
- [[Building an AI-Native Engineering Org (Gondala)]] — Gondala's autonomous-merge pipeline is Loop 1 (evidence-gated merging, risk-routed review) implemented and measured; Finch supplies the framework, Gondala the field data.
- [[Software Factories, Light and Dark]] — Finch's "runs dim, not dark" lands squarely in this debate, arguing against the dark factory without retreating to full human review.
- [[Full-Stack Verification Is the New Trend]] — Finch's widened PR gate (E2E browser testing, cross-repo checks, functional validation against the ticket) is a concrete instantiation of that source's verification-broadening argument.

---
*Sources: [[raw/andrew-finch]], [[summary/andrew-finch]]*
*Last updated: 2026-11-26*
