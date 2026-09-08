# Agents and Acquiring Debt

bl00cyb's essay (cowritten with Tom Henderson, fact-checked by Mark) argues that coding agents don't remove technical debt — they move it. Cheaper early information-gathering lets teams defer commitments to "the last responsible moment," which changes the *kind* of debt they accrue. The new kind is **comprehension debt**: the gap between what a system does and what anyone understands about *why*, accrued every time an agent's work is LGTM'd without being understood, and carrying a higher interest rate than the classic technical debt it resembles.

---

## Key Quotes

> "Debt is an instrument, not a smell."

The authors recover Ward Cunningham's original meaning: shipping an expedient-but-known-to-be-replaceable design *deliberately*, to reach a customer sooner. Against the "technical debt = code smell" folk reading, this is a useful correction — debt as a timed, priced financial decision rather than a moral failing.

> "Agents didn't create the why-problem; they universalized it. Every agent is the new hire on day one, forever."

The essay's sharpest line. The person who knew *why* the system is the way it is was always a single point of failure — the point is that agents make that failure the default state. Comprehension debt isn't a one-time loan; it's a revolving line of credit that re-opens every session.

> "They answer *what*, not *why*: they have the current state and the commits, not the folklore or the rejected alternative with reasoning."

The mechanism behind comprehension debt, stated cleanly. An agent can summarize what a commit did; it cannot tell you which alternative was rejected and why, because that reasoning never landed in the repo. This is the same asymmetry [[The Knowledge Chipper]] describes from the process side — context that evaporates with the session.

> "This might have the same fix as always: record decisions when you make them (with, say, ADRs), except now agents can help write the ADR at decision time, and agents are also its hungriest readers."

The prescription, and the most interesting twist: the ADR is no longer a human chore bolted on afterward but something an agent can produce *at the moment of decision*, and something other agents will actually consume. It turns the fix from documentation-theater into a machine-readable substrate.

> "Making a decision later is generally smarter because Future You knows more than Present You (supposedly), always, which contributes to determining the 'last responsible moment.'"

The "last responsible moment" is options thinking — the same logic as [[The Cost YAGNI Was Never About]] and [[YAGNI]] — restated as a *scheduling* problem rather than a thrift problem. AI pushes the moment later because the information that used to be expensive is now cheap to gather.

> "AI companies that sell coding agents are the equivalent of predatory credit card companies offering introductory cards on college campuses to freshmen."

Mark's metaphor, and the essay's most provocative sentence. It sits in real tension with "debt is an instrument, not a smell": if agent-acquired debt is *predatory*, it isn't being taken on deliberately at all — which the authors quietly concede up front by assuming debt is taken on intentionally "for this piece."

---

## Key Themes

- **#concept Comprehension debt** — the AI-native debt type: accepted-without-agreement, never surfaced, generated at high volume, and priced higher than technical debt because it compounds into a system nobody can explain. The closest existing page is [[Cognitive Debt]]; this essay adds the *mechanism* (what-not-why) and a *fix* (ADRs).
- **#concept The last responsible moment** — commitments deferred until further information can no longer change them for the better, disciplined by "holding costs" so deferral doesn't decay into procrastination.
- **#pattern ADRs as comprehension paydown** — record decisions at decision time; agents write them and read them. The catch the authors flag themselves: "dueling sycophancy banjos" — agents throwing each other under the bus when used to close the debt.
- **#concept Debt as instrument** — Cunningham's original, pre-"smell" meaning: expedient-with-knowledge-of-cost, chosen to reach a customer sooner.
- **#person Kate Chapman & Tom Henderson** — the conversation partners whose terms "genie wranglers" and "robot life coaches" frame the authors' starting point: humans managing AI rather than being replaced by it.

---

## Critical Analysis

**The real contribution is naming comprehension debt as a distinct debt class with its own interest rate.** The field has been circling this for a year — Osmani lists "comprehension debt" as one of three warning flags in [[Loop Engineering]], and [[Cognitive Debt]] says the same thing as "code has become cheaper to produce than to perceive." What this essay adds is the *asymmetry*: agents hold `what` (state, commits) but not `why` (folklore, rejected alternatives). That distinction is what makes the ADR fix click — you don't need to persist the whole mental model, only the decision points where `why` diverged from `what`.

**But the fix is thinner than the diagnosis.** The authors gesture at ADRs and then immediately flag the problem — "dueling sycophancy banjos about whether you or other AI is more important to pander to" — without resolving it. If the agent that writes the ADR is also the one being evaluated by the agent that reads it, the ADR is an incentive problem, not a documentation problem. [[Claude Is Not Your Architect]] makes the sharper version of this point: "'Claude designed it' is not an ADR," because the ADR's job is to record a *human* committing to a tradeoff.

**The "debt is an instrument" recovery is valuable but unstable.** Re-reading Cunningham is a genuinely useful corrective to the moralized "debt = smell" folk reading, and it grounds the essay's central claim that AI shifts *when* rather than *whether*. But Mark's predatory-credit-card metaphor then reintroduces the moral frame through the back door: freshman can't take on credit card debt "knowingly," and most LGTM'd agent work is exactly that kind of unknowing acceptance. The authors admit as much in the front-matter caveat ("we assume debt is taken on intentionally. This is often not the case") and then proceed anyway — which is honest, but leaves the essay describing the deliberate case while its sharpest metaphor describes the negligent one.

**"The last responsible moment" is the most reusable idea and the least developed.** It's options thinking with a scheduling frame, and the connection to [[Optimizing for Decision Points]] is strong: both say the scarce resource is *timing* human judgment, not producing code. But the authors dispatch the hard part in one paragraph — "holding costs" — when that's where the whole framework lives or dies. What's the cost of waiting? How do you know the information you're waiting for will actually arrive? [[Discovery Debt]] is the failure mode they're gesturing at and don't name: deferring a decision doesn't retire the assumption it rests on, it just lets that assumption compound invisibly.

**The essay's honesty about scope is a feature.** "Very much consider this open for feedback," "we said NO to that side quest," "unless of course our collective ADHD brains decide to fixate on something else" — this is a first pass at a series, flagged as such, and it reads better for it. The framing that "LLMs shift the *entire system*" and that pieces moving out of sync can leave you "further behind than you would have been without involving AI" is the thesis worth the promised follow-up.

---

## Related Pages

This source is the dedicated deep-dive on a concept several pages already flag in passing: **[[Loop Engineering]]** names comprehension debt as one of three warning flags and this essay is the argument for *why it's the highest-interest one*. It gives **[[Cognitive Debt]]** a mechanism (the what/why asymmetry) and a fix (ADRs) that page lacks. It names the *why*-loss that **[[The Knowledge Chipper]]** describes as context evaporation, from the artifact side rather than the process side. And it grounds **[[Optimizing for Decision Points]]**'s decision-timing thesis in a debt framework: the last responsible moment is where human judgment buys the most, and deferring past it is how comprehension debt compounds. [[Principal Drift]] names the *observable* half of the same debt — comprehension debt is the loss of understanding; principal drift is the loss of control it produces, surfacing as production incidents like Amazon's March 2026 outages — and supplies the routing framework (tier-1 review vs. systems inspection) for keeping agent work from being accepted-by-default. [[AI Handles Incidents, Engineers Lose Touch with Their Systems]] shows the same debt in a second domain, operations: every routine incident an AI SRE auto-resolves is practice the on-call engineer didn't get, so the debt lands in responders' muscle memory rather than the repo.

---
*Sources: [[raw/agents-and-acquiring-debt]], [[summary/agents-and-acquiring-debt]]*
*Last updated: 2026-08-25*
