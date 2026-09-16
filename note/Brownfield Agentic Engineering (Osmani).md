# Brownfield Agentic Engineering (Osmani)

Addy Osmani's essay on bringing coding agents into legacy codebases argues that the real work is making hidden constraints visible and cheap changes trustworthy — through a zone map that governs agent autonomy, characterization tests that lock existing behaviour, durable research artifacts, and a harness that accumulates every correction the team refuses to pay for twice.

---

## The argument in one paragraph

Osmani claims that the failure mode of agents in brownfield codebases is not model quality but missing structure: an agent thrown unsupervised at an old system produces code that "works" with the wrong design and brittle tests, and the fix is a graduated autonomy regime — zones drawn by humans, blast radius determining supervision, characterization tests before any improvement, migrations finished in complete units, and parallelism only after a single unit has a dependable judge. The claim is falsifiable: if zones, memos, and characterization tests are unnecessary ceremony, teams that skip them should see the same defect and rollback rates as teams that adopt them, and agent throughput in legacy code should already match greenfield throughput. His strongest empirical hint the other way is SWE Refactor Bench's "Blindness" result — across 520 agent runs on migrations, only 28 passed the audit — which suggests that without the structure, agents reliably produce migrations that look done and aren't.

## Key quotes

> the repository is no longer a complete description of how the thing actually behaves

The cleanest one-sentence definition of brownfield in the piece, and the premise everything else rests on: the gap between the code and the truth is exactly where agents are most dangerous, because they can only read the code.

> A person draws the map, not the agent; left to choose, the agent starts in the scariest file, because the scariest file has the most interesting names.

The best line in the article — an observed failure mode stated as a law. It quietly refutes the fantasy of the fully autonomous agent that figures out where to work on its own; interestingness is an attractor, not a priority function.

> Write down what the code can't say, and nothing else.

A disciplined counter-position to the markdown-stuffing era: agents infer the system map well enough on their own, so documentation effort should go exclusively to conventions, trade-offs, and history that are genuinely un-inferable. This is a testable claim about where context engineering effort pays off.

> Every repeated correction is a missing piece of the harness.

The conversion rule that makes the whole approach compound: a review comment appearing twice should become a lint rule, hook, type, test, or skill — prose only for what can't be enforced mechanically. The harness becomes "a record of failures the team has decided not to pay for twice."

> Agents have changed the price of trying several plausible implementations. They haven't changed the evidence required to choose one.

The economic thesis of the piece, and the reason CTOs are now running competing rewrites in parallel: generation got cheap, selection didn't. Everything downstream — review load, oracle quality, merge gates — follows from that asymmetry.

## Critical analysis

What's non-obvious here is the direction of the zone rule. Most agent-adoption advice optimises for where agents can go fast; Osmani inverts it and starts with where they must not go, and gives the red zone teeth ("human pairing on every step or the work not happening") rather than treating it as a warning label. The observation that agents gravitate toward the scariest files is the kind of thing you only write after watching it happen, and it justifies the whole zoning apparatus in one stroke.

The strongest section is the migration survey, because Osmani refuses the flattering numbers. He flags Asana's $12,000 Enzyme cleanup as "a vendor-reported cost of generation, not a controlled savings study," and notes that Stripe's 3.7M-line migration — the most impressive figure in the piece — involved no agents at all. The transferable lesson he extracts, that the durable artifact is the migration machine rather than the migrated code, is a genuinely useful reframe that survives the agent hype cycle intact.

The weaknesses are the usual ones for the genre. The zone map is asserted, not demonstrated: no example of an actual zone drawing, no account of what happens when teams disagree about where yellow ends. The SWE Refactor Bench statistic is dropped without enough context to evaluate it — 28 of 520 is striking, but we don't know what the runs were attempting or whether the structure Osmani prescribes would have moved the number. And the piece is silent on the political economy: who owns the zone map, what happens when the person who understands the red zone leaves, and how a team finds the slack to write characterization tests when the agent is supposedly making everything faster. The AOL homepage anecdote is charming and makes the "production traffic is the only real spec" point vividly, but it's also a story about heroics — the piece never addresses whether the zone regime scales to organisations that don't have an Osmani on call during his day off.

What's left out almost entirely is failure recovery: "recovery path" appears once in the parallelism section and is never developed. For a piece about old systems where things break, the absence of an incident-shaped section is a real gap.

## Related

- [[A Practical Guide to Brownfield AI Development]] — Pupius's guide covers the same problem (agents in legacy codebases lacking structural guardrails) and this source strengthens it with a concrete autonomy vocabulary: where Pupius argues for building guardrails incrementally, Osmani supplies the zone map and earned-promotion rule that make "incremental" operational.
- [[AI Code Migration with Claude Code]] — Anthropic's field report claims the fix-the-process thesis from inside one organisation's migrations; this source nuances it by surveying many migrations and concluding that what transfers is the structure, not the agent setup — and by adding the "complete units" rule that half-finished migrations actively mislead agents.
- [[Agentic Code Review]] — Osmani's own review guide argues the bottleneck has shifted to trusting code; this piece complicates that frame for brownfield specifically, where review attention must be rationed by blast radius and "the largest blast radius and weakest oracle" rather than by volume alone.
- [[Harness Engineering (OpenAI)]] — OpenAI's report treats the harness as something built deliberately ahead of agent work; this source nuances that picture for legacy systems, where the harness is assembled reactively from repeated corrections and becomes "a record of failures the team has decided not to pay for twice."

---
*Sources: [[raw/brownfield-agentic-engineering]], [[summary/brownfield-agentic-engineering]]*
