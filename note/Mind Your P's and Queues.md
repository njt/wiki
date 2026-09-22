# Mind Your P's and Queues

Daniel Vacanti's Craft 2025 talk takes the English expression "mind your P's and Q's" ("watch what you are doing") and turns it on agile's favorite ceremony: prioritization. His argument is that prioritization sets the start line while value is realized at the finish line, so the order that matters is governed not by the backlog but by queues — how many things are in progress at every level of the organization. A Monte Carlo simulation shows priority #15 beating priority #1 once WIP rises, and the flat verdict follows: "Prioritization is waste." The gist pairs the full transcript with a digest that candidly lists what the talk never answers.

---

## What the talk argues

- **Prioritization optimizes the wrong moment.** The Scrum Guide demands an ordered backlog to "maximize value" — and Vacanti stresses *maximize, not optimize*. But value is realized only at delivery, so "we really care about the order in which items finish," while prioritization "is all about figuring out when items start."
- **Two unverified assumptions hold agile up.** Frameworks assume items that start sooner finish sooner, and that items roughly finish in start order. His challenge to every audience: match your finish order against your start order. "In all the organizations I've ever been to, they have never done that."
- **Queues are everywhere and invisible.** "Up next," "analyzing," "creating," "validating" — every board column is a queue, and so is every level of the work breakdown structure: portfolio, initiative, feature, team. "You've got Q's coming out your R's."
- **WIP explodes across levels without anyone noticing.** A team pulling one story each from features A–E looks disciplined at story level but has pulled five features into progress. Multiply by "10, 20, 50 teams doing exactly the same thing" and you get an organization-wide explosion of work in progress — "you're massively, massively shooting yourself in the foot."
- **The Monte Carlo proof.** The customer's question is "When will it be done?" Monte Carlo simulation answers it — but only under two assumptions that "never happen": one thing at a time, strict priority order. At WIP=2, priority #9 already loses to #10. At WIP=9 (of 18 features), priority #1 has a lower completion chance than priorities 2–15. With everything in progress, "stories not assigned to a feature" beats #1 — nine features outrank the top priority.
- **The snowball.** Screw priority → why plan? → no predictability → no idea what products to invest in across the portfolio. "Pick any P word that you want in Agile and it will always come back to the fact that if you're not controlling queue size, doesn't really matter."
- **Q&A candor.** Team size is a red herring — he ran a 60-person standup in under 15 minutes. The Agile Manifesto has "no language around managing work in progress, no language around flow." Scrum and SAFe becoming synonymous with agile is its "biggest perversion." Lean and Agile are not either/or. And on several questions: "All of my answers have been 'I don't know'… You go figure it out."

## Key quotes

> "We really care about the order in which items finish… prioritization is all about figuring out when items start. Doesn't that seem backward?"

The whole talk in one move. It lands because it's checkable, not rhetorical: pull your own board data and see whether finish order correlates with start order at all.

> "It might look like you're doing all of the right things, but you're massively, massively shooting yourself in the foot because of this explosion of work in progress."

The hierarchy illusion: story-level WIP discipline is what teams can see; feature- and portfolio-level WIP is what actually determines flow. This is the talk's most transferable idea, because every level of a work breakdown structure hides the same trick one level down.

> "Priority number 15 has a higher chance of being finished than our priority number one."

The WIP=9 simulation result. Once most of the portfolio is in progress, completion probability is set by queue position and age, not by the ranking ceremony.

> "Stories not assigned to a feature has a higher chance of finishing than your number one, bank setup. If you're a product owner, what are you thinking right now?"

The comic-horror peak: after all the effort of prioritization, "it's just completely gone out the window." Unassigned work outranking the top priority is the reductio of pull-based overload.

> "The order in which that stuff gets done is now dictated by the number of items you're working on. It's not dictated by priority at all. Choose whatever order you want to. Doesn't matter because they're all at risk."

Note what this actually claims: not that order is wrong, but that order is irrelevant because everything is at risk. That's the version of the thesis the simulation supports — and it's narrower than the slogan.

> "Prioritization is waste."

The verdict, delivered flat. It is also the talk's most overreached sentence; see the take below.

> "Weighted shortest job first is a thing. It's a real thing. Safes implementation of weighted shortest job first is not a thing."

He offers to defend it in Q&A — nobody asks. The aside matters because WSJF is the scaled-frameworks' most quantitative-looking ritual, and the same WIP argument dissolves it.

> "You can have a team of 60 people as long as you're not working on 60 things."

His answer to the "ideal team size" question, backed by his own 60-person standup and Prateek Singh's 40-person team. The constraint lives in concurrent work, not headcount — the same relocation of the bottleneck [[Shaped by Demand — The Power of Fluid Teams]] performs on team structure.

> "The biggest perversion of the Agile Manifesto has been these frameworks like Scrum and Safe have become synonymous with Agile. Oh yeah, I'm doing Agile because I'm doing Scrum. Maybe you are, maybe you aren't."

From the Q&A. The Manifesto's real gap, for him, is structural: nothing about WIP or flow, which is exactly the half the frameworks never picked up.

## Key themes

- **#concept** — Start order vs. finish order: value is realized at delivery, so finish order is what "maximize value" actually requires; prioritization only governs starts. The empirical test — correlating the two orders — has never been run in any organization he's visited.
- **#concept** — The WIP explosion across the work breakdown structure: each level of the hierarchy multiplies concurrent work while each team's local view looks disciplined.
- **#pattern** — Limit WIP at every level; read board columns as queues. The visualization you already have is the queue map you're ignoring. Discover the organization's "optimal capacity" — "maybe it's one thing, maybe it's two things. I guarantee you it's probably not 18 things."
- **#tool** — Monte Carlo simulation for "when will it be done?" and "how many features make the date?" — with the caveat that the naive version assumes one-at-a-time and strict priority order, so model your actual WIP. Reading path: Little's Law (Dr. John Little's papers) for queues, Shewhart/Deming/Wheeler for variation, his own books in order 2, 1, 3.
- **#person** — Daniel Vacanti: kanban and flow metrics author (*When Will It Be Done?*, *Actionable Agile Metrics for Predictability*), heir to the Little's Law lineage, allergic to scaled-framework ritual ("SAFe's implementation of weighted shortest job first is not a thing").

## Opinionated take

The start/finish inversion is the best kind of conference argument: simple, correct, and falsifiable with data every team already has. His challenge — match finish order to start order — is a five-minute analysis that would embarrass most organizations, and the underlying queueing intuition (Little's Law) is not controversial. The hierarchy example is the real gem: WIP discipline that holds at one level while exploding at the level above is a failure mode no framework's ceremonies even name.

But the headline outruns the model. His own numbers show priority mostly intact at WIP=2 (only #9 vs #10 flips); the chaos arrives at WIP=9 and full-pull. The honest thesis is conditional — *uncontrolled WIP makes prioritization waste* — and he half-concedes it in Q&A ("be careful as you start getting into those higher numbers"). "Prioritization is waste" is a talk-title upgrade, and its corollary ("choose whatever order you want to") quietly erases the product owner role without saying what replaces it. The digest's sharpest catch goes further: his own admission that "only when we deliver to the customer do we actually know if something's valuable or not" dissolves not just Scrum's ordered backlog but *any* upfront value ranking, real WSJF included. He doesn't follow the thread, presumably because it would take his forecasting books down with it — "when will it be done?" is only worth answering if you've decided what's worth finishing.

The evidence is also weaker than the rhetoric: the damning reorderings come from a simulation he built, presented to an audience he has just chastised for never verifying anything. No field data, no before/after, no start/finish scatter from a real board. And the org-level gap is where the talk ends precisely where the problem begins: the diagnosis is organizational (50 teams × 5 features), but nothing is said about who owns a portfolio-level WIP cap, how it's enforced, or what happens when demand exceeds capacity — the questions [[Queues Don't Fix Overload]] answers for systems (identify the bottleneck, then back-pressure or shed load) but which no agile framework answers for organizations.

The agentic-era reading is why this talk belongs in this wiki now. Agents make starting work nearly free: every agent pulled onto a task is a feature pulled into progress, and a fleet of coding agents is a WIP-explosion machine with a prioritized backlog taped to its side. Vacanti's unscalable question — who caps concurrent work across 50 teams? — is now literally the orchestration question, and his correlation check is a ready-made metric for agent fleets: does finish order track start order at all, or is everything "at risk"?

## Related pages

- [[Queues Don't Fix Overload]] — Hebert makes the same move at system level that Vacanti makes at org level: the queue is the primary control surface, not an implementation detail. Hebert also supplies the cost accounting Vacanti skips — low WIP means idle capacity, and back-pressure (saying no) is the price of flow.
- [[Lean Software Production]] — Wynne's Lean pillar treats WIP control and flow as industrial safety equipment for the agentic era; Vacanti provides the quantitative demolition of the prioritized-backlog alternative that Lean's flow half replaces.
- [[Lean, Not Backpressure]] — kqr's single-piece flow prescription gets a nuance from Vacanti: optimal capacity may be 2–4, not 1, and the start/finish correlation check is the empirical test of whether your batch size is actually doing anything.
- [[Shaped by Demand — The Power of Fluid Teams]] — the sibling Craft 2025 talk with a convergent constraint: North fixes demand and varies team composition, Vacanti fixes capacity-as-concurrent-work and varies headcount not at all; both relocate the binding constraint from people to the system of work.

---
*Sources: [[raw/mind-your-p-s-and-queues-daniel-vacanti-craft-2025]], [[summary/mind-your-p-s-and-queues-daniel-vacanti-craft-2025]]*
*Last updated: 2026-09-13*
