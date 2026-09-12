# Continuous AI

Don Syme's term (with GitHub Agentic Workflows as the reference implementation) for a third category of software automation alongside CI and CD: the *subjective*, non-deterministic work — triage, labeling, documentation, perf, duplicate detection — that must run continuously inside a bounded context but has no binary pass/fail outcome. The thesis is that AI's next shift is from individual chat productivity to situated, collaborative automation folded into the change-driven rhythm software teams already run on.

---

## Key Quotes

> "We don't say we're putting AI into your CI. No, no, no, no, no, no. We don't do that at all. It's CI CD, they stay as they are. And continuous AI is a separate bucket of things."

Syme's hardest line, and the one that does the most rhetorical work. CI/CD is "the place where the grown ups are" — the determinism that establishes software actually works. Positioning Continuous AI as a *peer category* rather than an intrusion is how he defuses the visceral "don't let AI near my pipeline" resistance. Whether the boundary is real or just diplomatic is the open question: the same repo, the same GitHub Actions substrate, the same human-in-the-loop at the PR merge point. The bucket is separate; the plumbing isn't.

> "The stronger the train tracks, the faster you can run."

The inversion that separates this from most agentic hype. Guardrails aren't a speed limit on autonomous agents — they're the precondition for running them unattended. "You've got to be able to sleep at night." This is the same thesis as [[Guardrails and Feedback Loops]] (deterministic enforcement beats instructions), restated for the operations modality: you can only turn agents loose overnight if you trust the constraints, not the model.

> "My job as a factory creator is to deliver high quality pull requests where the reviewer is equipped... My job is to equip the reviewer with all the information they need."

The reviewer is the bottleneck, so the factory's real product is *equipped review*, not code. In Syme's CI-performance example the evidence is the before/after timing delta, banked right there in the PR. This is a sharper version of the human-in-the-loop that [[Cloud Software Factories]] sketches: not just "a human approves," but a human who's been handed evidence, risk, and trade-offs so the approval is cheap and trustworthy.

> "If it produces rubbish, you throw it away. You know, the can, it's got a crush on it, you know, throw it away."

The factory economics made vivid: AI work is cheap, so discard is fine — *if* your quality gates catch the trash before it reaches the reviewer. Syme is explicit that "a lot of the work is on the quality gates." The unexamined assumption (that trash is *obviously* trash) is the weakest joint in the whole argument — see below.

## Key Themes

- #concept — **Continuous AI** as the subjective complement to CI/CD, bounded by the repo
- #tool — **GitHub Agentic Workflows**: bring-your-own coding agent (Claude Code, Copilot CLI, Gemini CLI) with guardrails, open source, public preview
- #pattern — **RepoAssist**: one broad-spectrum workflow over an agent zoo, for cost control and attention management
- #pattern — **examination testing**: run the same work with multiple models *simultaneously* to compare them (sequential runs leak answers through the shared ledger)
- #pattern — **quality gate stacking**: pile deterministic gates before the human so the reviewer never sees trash
- #pattern — **goal chasers**: a slow, cooperative daily loop — one PR toward a goal per day, rather than a thousand at once
- #person — **Don Syme**, GitHub principal researcher, the frame's originator

## Critical Analysis

**The frame earns its keep.** "Continuous AI" names a real gap in the conversation. Individual-productivity chatter and loop engineering both live in the single-developer frame; Syme is right that the missing question is *how does AI automation live in the shared, bounded context where teams actually work*. CI/CD gives him a template people already understand, and the "bounded context" argument — AI goes off the rails without a firewall around inputs and outputs — is genuinely the load-bearing claim. The repo-as-bounded-context maps cleanly onto the organizational empowerment story: CI/CD succeeded because it gave developers permission to claim limited cloud resources without asking the cloud team.

**But the factory analogy has a hole in it.** "If it produces rubbish, you throw it away" assumes rubbish is recognizable — the crushed can is visibly crushed. AI's failure mode is the opposite: plausible-looking-but-wrong work that passes CI, passes style gates, and gets merged. That's the review problem the transcript never touches, and it's the same gap [[Harness Engineering is not Enough]] names against lights-off factories: subtle degradation plays out over months, invisible to any per-run quality gate. Syme's gates catch the crushed cans; they don't catch the can that *looks* fine but is 2% too small.

**Human review doesn't scale, and the transcript knows it.** Host Guy Pigiani pushes back hard: "no auto-merge" is "an untenable destination" when the factory is producing more PRs than a team can review. Syme's answer — "we're being responsible," plus automated pushes to PR branches and one-long-running-PR tricks — is honest but not a path. This is the exact pressure [[Cloud Software Factories]] frames as COGS-not-R&D, and neither Syme nor Lloyd resolves it.

**Cost control is the sharpest, most portable claim.** Syme's line that cost budgeting is "the thing that's missing from the harness discussion" lands precisely on [[Building an Advanced Agentic Harness]] — a tutorial whose `BudgetMulti` primitive is the exception that proves the rule. Per-run cost visibility and examination testing are the features that would make the factory a *financial* object, not just an engineering one, and that's where the enterprise adoption case actually lives. [[Platform Engineering as the AI Control Plane]] is the org-chart version of the same need: someone has to own model approval and cost governance.

**Loop engineering is orthogonal, not competing.** Syme treats [[Loop Engineering]] / [[Designing Agentic Loops]] as adjacent-but-different: the concepts "flow very freely" into Continuous AI, but the ledger of record differs (repo + issues + PRs vs. a local dev loop). The "goal chaser" is the bridge — a loop, but slow and cooperative, one PR a day toward a measured goal rather than a thousand dumped on review. That pace-shaping is Continuous AI's most underrated idea: it's not about doing more, it's about making automation *reviewable* by tuning its cadence to human capacity.

**Verdict:** the category is useful and the implementation is real, but the talk is stronger on taxonomy than on failure modes. Continuous AI explains *where* AI automation should live and *how* to bound it; it's conspicuously quiet on what happens when bounded, gate-passing automation is subtly wrong. That's the frontier [[Lean Software Production]] points at — the work shifts from producing code to engineering the system that produces it, and the hard part is judging the system's output, not producing it.

---
*Sources: [[raw/022cc46bcfca6d72131b198e9075ba0c]], [[summary/022cc46bcfca6d72131b198e9075ba0c]]*
*Last updated: 2026-08-26*
