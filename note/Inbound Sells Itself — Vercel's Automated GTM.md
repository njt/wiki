# Inbound Sells Itself — Vercel's Automated GTM

A Theory Ventures office-hours interview where Vercel COO Gene Grosser walks through automating Vercel's entire inbound sales motion with a single-agent system: one engineer at 20% time, a human-in-the-loop QA flywheel for six weeks, then full autonomy — ~90% of inbound processed for ~$1,000/year of inference. The real payload is the design arc: prompt → rules-in-code → model-for-judgment-only, which is one of the cleanest public articulations of how a production agent system should mature.

---

## The arc in four moves

1. **Encode the best practitioner.** The prompt was written from the team's #1 SDR. "Sales is a skill. It's actually not the most well codified skill on the planet" — so documenting your own best practice *is* the work: "a big part of your alpha is going to be, do you have a well developed point of view on the sales journey?"
2. **QA as teaching, not inspection.** Every agent output went to a Slack channel where the top SDR marked up qualification and edits. Six weeks later she "was very infrequently disagreeing with the agent" and was pulled out.
3. **The prompt rots.** As Vercel moved upmarket the 125-line prompt became 1,000+ lines, and "if it's just a prompt, the model actually won't always follow some of the rules." The fix: 14 deterministic rules in code, model used only "for thinking exactly where a human would have thought previously."
4. **Supervise like a manager.** Read 100% of a new hire's first hundred emails, then sample 1-in-100 forever. Plus a watchdog agent that detects rule breaks and undoes them — an escalation agent "just like a manager would."

## Key quotes

> "We started out with about 125 line item prompt... then it became over a thousand lines. And now what we've done is actually realize that was pretty darn problematic because one, if it's just a prompt, the model actually won't always follow some of the rules."

This is the anti-prompt-debt thesis stated from the buyer's side of the org chart, not the engineering side. When business rules live in prose, compliance is probabilistic; when they live in code, compliance is total and auditable ("which rules fired" beats "what did the model do").

> "We basically took 10 people down to about 1000 bucks of inference plus infrastructure spend."

The headline unit-economics number. Note what it leaves out: the year of senior human calibration that made autonomy safe. The $1,000 replaces the *steady state*, not the build.

> "Covid turned BDRs kind of into email marketers... the value of humans is talking to humans."

The labour story is not elimination but redirection: agents absorb low-intent lead work that was never worth human calories, and humans concentrate on conversations — the finite resource being "number of hours in the day that I can talk to another human."

## Themes

#concept #pattern #tool

- **Rules in code, judgment in the model** — the same design discipline as [[A Convention Is Not a Constraint]] and the guardrails literature, arrived at independently from a COO's chair.
- **Human-in-the-loop as a temporary training phase**, with sampling as the steady-state QA regime — kin to [[Feedback Loop is All You Need]].
- **The org chart absorbs the automation**: SDRs promoted to outbound rather than cut, and the same playbook extended to MQLs.

## Analysis

What makes this source unusually valuable is that it is a *business* person describing what engineers usually describe, and he gets the engineering right. The escalation-agent pattern — a second agent that watches for rule violations and reverts them — is exactly the linter-over-instructions architecture the guardrails world keeps proposing, shown working in revenue-critical production.

The honest caveat is in the framing Grosser never states: the system's reliability was *purchased* with a top performer's six weeks of full-time markup, and the "point of view on the sales journey" that makes the rules possible is an organizational asset most companies don't have. Anyone hearing "$1,000 replaces 10 people" without the preceding 45 seconds is hearing the wrong lesson. The comparison with [[The Harness Is the Company]] is direct: Vercel did not buy an AI SDR product; it built a harness around its own codified best practice — and the codification, not the model, is the moat.

There is also an unresolved tension in "the value of humans is talking to humans." Grosser's own AE-prep vignette — an exec taking three CTO calls by 11am because an agent wrote the briefs — extends agent leverage *into* the human conversations too. The endpoint is not humans free of agents; it is humans as the face of an agent-run pipeline.

## Relation to the wiki

- [[The Harness Is the Company]] — this is the ladder-climbing argument with a concrete rung documented: Vercel's inbound function *is* a company-internal harness around its own best practice, and owning it is the competency.
- [[Feedback Loop is All You Need]] — strengthens the claim with a revenue-side case: the Slack QA flywheel is the loop, and the six-week shutdown point is a measured answer to how long human-in-the-loop should last.
- [[A Convention Is Not a Constraint]] — complicates it from the opposite direction: here the move *to* code comes from an executive, showing the prompt-vs-rules distinction is now legible outside engineering.
- [[AI in the Firm — Bottlenecks in Software Production]] — a useful contrast: where Chen & Stratton found gains absorbed by a review bottleneck, Grosser shows the same displacement logic in sales — human effort migrates up the intent ladder rather than disappearing.

---
*Sources: [[raw/virtual-office-hours-when-inbound-sells-itself]], [[summary/virtual-office-hours-when-inbound-sells-itself]]*
*Last updated: 2026-10-09*
