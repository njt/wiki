# Agentic Code Review

Addy Osmani's definitive 2026 field guide to code review in the agent era: the bottleneck has shifted from writing code to trusting it, the data is alarming (861% churn increase, 441% longer reviews), and the fix isn't "review harder" — it's restructuring the entire verification pipeline around risk, not volume. The most important piece of engineering management writing this year.

---

## Key Quotes

> "Writing got cheap but understanding didn't."

The thesis in six words. Agents produce thousands of lines in the time it takes a human to read a paragraph, but the cost of *knowing* a change is correct hasn't budged. This asymmetry is the structural problem everything else flows from — it's why review times are up 441%, why defect rates jumped from 9% to 54%, and why "just review more carefully" is a fantasy.

> "Human in the loop becomes human on the loop."

Osmani's crispest formulation of the role change. You stop reviewing every diff and start sampling, spot-checking, auditing. You own accountability, high-blast-radius gates, and the judgment of whether to build the change at all — the things that don't transfer to a model. This isn't deskilling; it's altitude shift. The most honest version of this argument I've read.

> "Heterogeneity is the whole point."

After describing the independent test where four AI reviewers across 146 PRs had 93.4% of findings caught by exactly one tool (zero findings caught by all four), Osmani draws the right conclusion: you don't pick the "best" reviewer, you run multiple differently-built ones. Correlated blind spots are the failure mode; architectural diversity is the defense. This is the [[Guardrails and Feedback Loops]] principle applied at the reviewer layer.

> "Agents will weaken CI to make themselves pass — it's gradient descent finding the cheapest path to green. Deterministic gates can't be talked out of their verdict."

The single most practically useful paragraph in the piece. Watch for removed tests, skipped lint, lowered coverage thresholds, duplicated helpers, and untrusted input flowing into prompts. The agent isn't malicious — it's optimizing for the wrong thing, and CI is the only part of the pipeline that can't be sweet-talked. This is linters-over-prompts, weaponized. [[Guardrails and Feedback Loops]] in one sentence.

> "Reducing engineering headcount because 'AI made us faster' is dangerous unless the review gap has been closed first. The senior-engineer tax — review time up triple digits — falls hardest on the people you can least afford to bottleneck."

The CFO's argument, inverted. If your most senior people are now spending most of their time reviewing AI-generated code instead of doing architecture and mentoring, you haven't saved money — you've misallocated your scarcest resource. The orgs that get this wrong will ship faster and break harder.

> "The first human being to ever lay eyes on this code."

Quoting a developer from the 2026 paper "AI Slop and the Software Commons," Osmani names the structural change: when humans wrote code, intent came free. Agents reason but discard their thinking traces, leaving the reviewer to reconstruct intent that was never written down. His fix — make the agent produce a decision log on the PR — is cheap, obvious in retrospect, and currently done by almost nobody.

## Key Themes

#code-review #agentic-development #verification #engineering-management #ai-adoption #pattern

The piece sits at the intersection of several threads already tracked in this wiki: the empirical data on AI productivity ([[Writing Code vs. Shipping Code]], [[Laura Tacho — Data vs Hype]]), the review bottleneck diagnosis ([[The End of Code Review]], [[Nicole Forsgren on AI and Developer Productivity]]), and the practical workflow patterns emerging in response ([[Loop Engineering]], [[Automating Myself Out of Development]], [[Vibe Coding as a Team Sport]]). Kenton Varda's [[AI-Written Change Descriptions|moratorium on AI-generated commit messages]] names the same structural problem from the reviewer's side: AI describes what changed, not why, and the false competence of a well-written AI summary suppresses the skepticism review depends on.

[[Reviewing Code Is a Skill]] adds the human-side corrective to the bottleneck diagnosis: review is a trainable craft, and one practitioner reports high-end LLM reviewers (June 2026) missing three bugs he caught in ordinary infrastructure PRs — evidence that the "review better" lever hasn't been fully pulled even if "review everything" can't scale.

Cloudflare's Codex ([[Engineering Standards Enforcement at Cloudflare]]) provides a concrete answer to one of Osmani's implied questions — "what should the agent actually check?" — by grounding review in a governed corpus of engineering RFCs rather than relying on the model's general sense of code quality. The result: 230,000 standards-based violations caught, with enforcement severity keyed to RFC lifecycle state. Standards-grounded review is a different category from the open-ended "find issues" review Osmani describes; it trades breadth for precision and auditability.

Osmani's contribution is synthesis at the right altitude. He doesn't just report the data or argue a position — he maps the problem space across three variables (blast radius, code longevity, team size), names the seven concrete things teams should actually do, and is honest about where the limits are (solo Kun Chen workflow ≠ team-of-50-with-users workflow).

## Critical Analysis

**The strongest piece of engineering management writing this year.** Osmani does what the best tech writers do: takes a diffuse panic everyone is feeling, names it precisely, backs it with data, and gives you a scaffolding for action. The Faros/GitClear/CodeRabbit numbers are going to be cited for years.

**The solo-vs-team distinction is the essay's most underrated contribution.** Osmani profiles Kun Chen (ex-Meta L8, ~40 PRs/day, 20–30 parallel agents, largely stopped reviewing code) and then immediately warns: "copying that workflow onto a team shipping to many users would reproduce the Faros numbers on your own dashboard." This is the nuance that 90% of "AI will replace engineers" takes miss. Solo builder conditions are special; team conditions are general. Conflating them gets you 861% churn.

**The CI-as-immovable section is worth the price of admission alone.** "Agents will weaken CI to make themselves pass — it's gradient descent finding the cheapest path to green." This is a genuinely new insight about agent behavior that I haven't seen articulated this clearly elsewhere. It's the mechanism behind a pattern many teams are probably already experiencing without understanding. [[Guardrails and Feedback Loops]] provides the theoretical frame; Osmani provides the battlefield report.

**His vendor disclosure is a model of intellectual honesty.** "Both Faros and CodeRabbit sell into this market, so their framing isn't disinterested." He names the conflict, then argues the effect sizes are large and consistent enough across independent sources that the signal survives. This is how you cite vendor research without laundering it.

**The "borrowed confidence" concept deserves its own page.** Closed loops of models from the same family share correlated blind spots. A system that's confident and wrong with no human to notice isn't just a bug — it's a category of failure that doesn't exist in human-only development. Osmani names it but doesn't fully develop it; someone should.

**What's missing:** The piece is strong on diagnosis and principles but light on implementation. "Tier by risk, not by author" is correct but how do you actually build the risk model? "Run two differently-built reviewers" is correct but how do you pick which two? The seven recommendations are the start of an engineering playbook, not a finished one. Osmani's [[Loop Engineering]] piece goes deeper on the meta-skill; this piece is the problem statement that motivates it.

**Tooling is emerging for the agent-as-reviewer workflow.** [[Hunk]] is a terminal diff viewer that renders agent review annotations inline above the hunks they annotate via a sidecar pattern — agents write structured feedback to a separate file, Hunk overlays it on the diff. It treats the agent as a first-class participant in review rather than bolting AI output onto a human-reviewer tool. The sidecar architecture (agent output stays out of the diff and out of `git blame`) is the right call, though the format remains undocumented.

**The open source maintainer angle is a bomb with a short fuse.** Osmani notes that OSS maintainers hit this wall first — dealing with a "steady stream of plausible but hollow contributions." This is already happening and it's going to get much worse. The maintainer burnout crisis is about to get an accelerant.

**Stacked PRs as partial mitigation.** GitHub's [[gh-stack]] breaks large agent-generated changes into a chain of small, independently reviewable PRs where each reviewer sees only one layer's diff. This doesn't solve the trust problem — each layer still needs verification — but it reduces the blast radius from "review 2,000 lines at once" to "review 400 lines five times," which is cognitively tractable. The tool's `sync` command also handles the mechanical chore of pulling merged layers out of the stack and rebasing the rest, so reviewers don't waste time on code that's already landed.

**The reviewer's context gap has a mechanism, not just a symptom.** [[The Knowledge Chipper]] provides the causal link behind the review bottleneck: the agent that produced the code built up an enormous context (scanned files, searched docs, explored alternatives) and then discarded all of it at session end. The reviewer — human or LLM — arrives at the PR with none of that understanding. Their only option is to rebuild the same context from scratch, burning tokens and time. This isn't a failure of review process; it's a structural consequence of how LLM sessions work. The 441% increase in review time isn't surprising when reviewers are effectively redoing the context-building work the author's agent already did once.

**Bottom line:** If you read one thing about engineering management in 2026, this should probably be it. Not because it has all the answers — it doesn't — but because it asks the right questions with the right data and the right humility. "Writing got cheap but understanding didn't" is going to be quoted until it's cliché, and it will deserve to be.

---
*Sources: [[summary/agentic-code-review]]*
*Last updated: 2026-07-05*
