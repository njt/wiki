# The Decoherence Trap

maevyn's essay introduces **decoherence** — the accumulation of locally valid but mutually incompatible models within a system — as a third form of technical debt, worse than infrastructural or advancement debt because it is created by *how* transitions are done and compounds exponentially. It uses that lens to argue both that AI coding agents are currently a decoherence accelerant, and that decoherence rules out a fast, isolated Software Intelligence Explosion — leaving us headed not for runaway superintelligence but for **Decoherence Collapse**, a software ecosystem we can no longer maintain without the very agents degrading it.

---

## The argument in brief

- **Three debts, one trap.** Infrastructural debt (pipes wear out) and advancement debt (standards move on) are familiar. The third arises from *patch-style* transitions: replace each failing pipe one at a time and you get a patchwork needing an expert in *both* systems — and with three or more systems, complexity goes combinatorial.
- **Grows with change, not time.** A decoherent codebase left alone for ten years is no worse. But "if you're making a lot of changes really quickly, you can also grow decoherent just as quickly."
- **The pressure spiral.** Under deadline pressure, experts stop the analysis early and do something they're "pretty sure" will work — which makes the next change even harder, which produces even sloppier work. Exponential growth of complexity from purely local decisions.
- **The Sweet Spot.** A simulation of actors mutating a graph shows an emergent curve: too little coherence and feature time explodes; too much perfectionism and iteration slows; a small band in between is where healthy systems live. The author concedes the weights are arbitrary — the point is that the curve can *emerge* from simple local rules.
- **Humans are already bad at this.** Most companies cycle between decoherence crisis, partial cleanup, relapse. Apple's culture of coherence is the rare exception, not the rule.

## Key quotes

> "It's what happens when slop drowns out the signal in an endless sea of noise."

The TL;DR of the whole concept — decoherence as a signal-to-noise failure in the models a system carries, not just in its code.

> "Decoherence does not grow over *time*, but with *change*."

The most operationally useful claim in the essay, and the source of its fragile optimism: if decoherence only moves when the system moves, stopping is always an option.

> "Code is not the textile, but the textile-machine."

The pivot against the industrial-automation analogy. Nobody buys code; they buy applications. So the recursive-self-improvement dream is not faster looms but "a machine that *designed* textile-machines" — unprecedented, and subject to entirely different failure modes.

> "We *cannot* make an AI that improves software *coherence* beyond the human ceiling without leaving the bubble."

The core anti-SIE proof. Coherence is tested by how an abstraction holds up across contexts *and across time* — future software, evolving requirements, laggard ecosystems. Both tests require round trips to reality, so an in-bubble RL environment can optimize speed and size while drifting catastrophically on the thing that makes software good.

> "I am far more afraid of it than runaway super-intelligence, because it's actually happening right now, in front of our very eyes."

The stakes claim. Whether or not one buys the SIE critique, the observable present-tense risk is dependency on agents that add more complexity per change than they remove.

## Themes

#concept (decoherence as a named failure mode) #pattern (the pressure spiral and the Sweet Spot) #tool (the graph-mutation simulation) #person (engagements with Tom Davidson, Cory Doctorow, Terence Tao)

## Analysis

The essay's best move is giving a name to something every senior engineer has felt in their hands: the codebase where each fix is locally defensible and the whole is unintelligible. Framing it as a *third* debt category is genuinely clarifying — refactoring budgets and dependency upgrades both leave it untouched, because decoherence lives in the joints between paradigms, not in the paradigms themselves. The "grows with change" insight also neatly explains why the AI moment is dangerous in a way mere sloppiness never was: agents are change-rate amplifiers.

The SIE critique is the more ambitious half and the more debatable. The claim that coherence cannot be measured in the bubble is strong, but it leans on an idealized picture of human software development — as if human teams validated coherence against reality reliably. The author's own inventory of studies (Harvard, CMU, the 300k-commit audit) shows humans producing decoherence at industrial scale for decades. If agents merely match the average engineering culture, the collapse argument still lands; but the essay needs agents to be *worse*, and the cited evidence shows "not clearly better," which is a thinner reed. The Bubble itself is also improving fast — real-world codebases as RL environments are exactly the round trip the essay says is impossible, just slow rather than absent.

The Omarchy / open-source coda is the most original detour: coherence as a *commons* that slash-and-burn re-customization of every dependency depletes, with the open-source process as history's most successful coherence engine. That's an argument worth its own essay (promised as *Open Capital*).

The remedy is quietly radical in 2026: finish the migration. Don't run two paradigms forever. The humblest advice in the piece is the hardest to follow under pressure — and the pressure spiral, not the model, is what agents are currently feeding.

## Related pages

- [[AI in the Firm — Bottlenecks in Software Production]] — the Harvard study the essay leans on (30% more code, 49% longer reviews, no output gain) is ingested here in full; this essay is the mechanism-level explanation for its bottleneck finding.
- [[Coding Agents and Complexity Budgets]] — both argue agents increase structural complexity per change; decoherence names *which* complexity: incompatible local models, not just more code.
- [[AI Slop Starts with the Codebase Itself]] — compatriot diagnosis: agent output quality is bounded by the coherence of what it lands in, so decoherent codebases compound slop into a feedback loop.
- [[Cognitive Debt]] — the human-side mirror: decoherence is what cognitive debt looks like from the system's perspective; both grow silently and are paid down only by deliberate, unrewarded work.
- [[Migrations — The Sole Scalable Fix to Tech Debt]] — shares the essay's insistence that transitions must *complete*; decoherence is precisely the state produced by migrations left half-done.

---
*Sources: [[raw/decoherence]], [[summary/decoherence]]*
*Last updated: 2026-10-21*
