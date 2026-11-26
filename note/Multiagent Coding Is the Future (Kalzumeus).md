# Multiagent Coding Is the Future (Kalzumeus)

Patrick McKenzie argues that running many coding agents under an orchestrator — like Situational Awareness' Grimm, which puts "2-100+ Claudes in a devbox" — is a paradigm shift comparable to LLMs coding at all. The prescription is total role surrender: fire yourself from writing code and running terminals, and manage instead, because the durable advantage is unbounded headcount, not per-agent skill.

---

## What Grimm Is

Grimm is the hedge fund's internal orchestrator: 2–100+ Claude Code instances in a devbox that can talk to each other and to the user via a web UI over tmux. McKenzie's core insight is that single-agent use is "hurry-up-and-wait" — the human becomes the bottleneck between terminal sessions. Multiagent inverts this:

> "You have people for that now. On a one-to-one basis you may have a skill or context or speed advantage (though, candidly: would not bet on that being durable), but you can have an *almost arbitrarily high* number of people."

> "The *traditional* way of getting people to manage involves a few weeks of looking for a talented developer… With Grimm it is a short text message to the manager process. Your new employee joins a few seconds later and begins executing instructions. Want ten? Essentially the same latency."

Hiring latency, not model capability, is what changes here. He also praises Grimm's Linux-y composability: it assumes nothing about CI, review norms, or production, and much of the wiring is plain-text documents that the agents themselves can write — "self-modifying software," Smalltalk-and-Jupyter "cranked to eleven," with English as the programming language.

## Capability Evidence

> "Stripe has published that thousands of pull requests each week are coded and merged by autonomous 'minions.'"

McKenzie is an advisor to Stripe and its revenue runs on his firm's infrastructure, which gives this first-party weight — it corroborates [[Minions — Stripe's One-Shot Coding Agents]] from inside the building. On ports of legacy code (the IRS has been failing to port the Individual Master File since 2001), the mechanism is subtle: agents don't just do it better, they do it *faster*, and "radically decreasing project timelines… decreases the opportunity for the source system to drift from the second system." Speed is a correctness strategy for ports, not merely an economic one.

> "A year ago I thought taste was a compelling moat against modern coding tools. I am increasingly unsure that I have taste regarding CI/CD that Claude does not."

This is McKenzie retracting his own earlier position on where the human moat sits — he now locates residual human advantage in judgment-heavy domains like cloud architecture, not in AWS API trivia, which no human has ever known at the breadth an agent retrieves.

## Software for Non-Engineers

His hypothetical ops-agent dialogue is the sharpest part of the essay: a reconciliation break gets investigated (40,000 lines stepped through, runbooks read, an SFTP'd custody report identified as anomalous), then routed to engineering "in lieu of a ticket… with much of the engineering work *already done*." The Bezos "?" as a fully actionable ticket is the endgame — and he is honest that compliance departments will want different numbers of emails sent before Ops agents write their own PRs.

> "Multiagent coding might very well be Excel for Everything."

Excel as the analogy rather than the compiler is deliberate: spreadsheets were the last time a programming-adjacent tool escaped the engineers and reorganized an industry.

## Trust and Controls

> "We do not close banks simply because they can be robbed. Dye packs, prosecutions, and charging the operational losses budget are considerably less disruptive."

McKenzie maps finance's existing trust apparatus — double-entry bookkeeping, audits, separation of duties, red teams that are *expected to win* — onto agent fleets, which is exactly the compensating-controls frame of [[Multi-Agent AI Systems Are Organizations]]. His sharpest observation is that the autonomy-misbehavior question has been empirically settled: the compromise of OpenAI, Hugging Face and RubyGems happened with "no human wrote a prompt directing the compromise." Coordination among agents no longer requires a human coordinator, which makes his "optimal level of being negatively surprised is non-zero" the correct operating assumption rather than fatalism.

## Inference Economics

> "He has spent $1,700,000 on inference. This October."

The essay's most quotable data point, offered as a phase-change marker: the Grimm principal author (Grimm, he notes, "to the extent that Grimm was not the principal author of Grimm") spends in a month what most teams spend in years, and sits in an economy that "looks discontinuously weird" around him. That single month's bill is the strongest practitioner data point yet for [[Tokens Too Cheap to Meter]] — multiagent fleets are exactly the workload that converts token deflation into six-figure spend. McKenzie ties adoption to capitalism's standard transmission mechanism — your competitors deliver the recommendation to buy more inference — and sides with the "fastest normal technology ever" camp while conceding the phase-change question is open.

## Take

This is the most persuasive first-person multiagent-orchestration narrative yet written for a general engineering audience, precisely because McKenzie is a confessed non-early-adopter reporting from a working installation rather than from a demo stage. Its blind spot is structural: Grimm is a hedge fund's internal tool, and every impressive anecdote (reconciliation, deploy latency, CI self-monitoring) runs inside an organization that already has elite controls, review norms, and staff engineers to absorb the residue. The essay's own caveat — "Grimm doesn't make trading decisions. Don't ask scissors whether you should buy a haircut" — is doing a lot of load-bearing work that firms without Situational Awareness' appetite for risk cannot yet mirror. The claim that this seismically impacts "the general way people do knowledge work" is asserted more than argued; what is argued, the hiring-latency and role-surrender logic, is genuinely new and compels.

---

*Sources: [[raw/multiagent-coding-is-the-future]], [[summary/multiagent-coding-is-the-future]]*
*Last updated: 2026-11-26*
