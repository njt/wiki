# Nobody Could Have Written the Ticket

James Randall — forty-two years a programmer, solo builder of a WebGPU 4X strategy game — argues that the dominant model of agentic development (backlog in, tickets to agents, PRs out) quietly assumes the ticket is a complete description of the work, when it is actually the residue of decisions made by someone who hadn't yet seen the thing running. The essay matters because it locates the failure not in tool quality but in the control structure: ticket-to-merge is waterfall with the cheating removed.

---

## The argument in one paragraph

Ticket-driven agent pipelines fail on design work not because agents implement badly but because they implement *faithfully*: a ticket is a hypothesis written before the thing existed, and an agent that satisfies it as a contract will never come back and say the hypothesis was wrong. The claim is falsifiable — if a meaningful fraction of design-quality feedback could be captured in acceptance criteria written in advance, or if machine evaluation could substitute for human contact with the running product, the ceiling Randall describes would not exist. He bets it does exist, that it covers well under half of most product backlogs, and that the real gain from agents is cheaper iteration (more hypotheses tested and killed), not throughput of closed tickets.

## Key quotes

> A ticket isn't the work. It's the residue of a decision, or series of decisions, somebody already made.

The reframe the whole essay hangs on: the expensive part — weighing options, discarding worse ones — happened before the ticket was written, and what remains is "the conclusion with the reasoning boiled off."

> The agent's problem isn't that it might get it wrong. It's that it gets it right.

The sharpest inversion in the piece. A faithful implementation of a bad spec looks identical to a faithful implementation of a good one — same green tests, same clean diff — so the absence of the pushback channel is invisible in the artefact.

> If we allow agents to run from ticket to merge then we've built the first faithful implementation of waterfall anyone's managed, and faithfulness turns out to be the flaw.

The historical point is doing real work here: waterfall was survivable because humans cheated — corridor conversations, quietly renegotiated requirements. An agent doesn't cheat, so the process's missing return path finally gets exercised as written.

> A specification precise enough to be unambiguous is isomorphic to the program. Push the ticket-writing far enough to make the agent reliable on design work and you have written the code in a worse language, more slowly, with no compiler.

This closes off the standard "just write better tickets" rebuttal with an economic argument, not a capability argument: the fix is self-undermining.

> If you're measuring tickets closed you'll conclude these tools are working brilliantly while the product gets worse. The thing to watch is how many hypotheses you tested and how many you killed.

From the three closing claims Nat flagged as the load-bearing section — and it is the one with the most immediate operational bite, because it names a metric (hypotheses tested and killed) that almost no team currently tracks, and implies the dashboard everyone does track is actively misleading.

## Critical analysis

The three closing claims deserve the weight Nat gives them, because they are where the essay stops being an anecdote and becomes a testable model. The strongest is the third — "somebody has to keep playing the game" — because it is nearly tautological: if the acceptance criterion for design work lives in a human head and often doesn't exist even there, then human contact with the running thing is the only place feedback can originate, and it bounds everything. The second claim (value as iteration count, not throughput) is the most actionable and the most falsifiable: it predicts teams optimising tickets-closed will ship more and learn less. The first — the specifiable fraction as a ceiling "well under half" — is the weakest, and Randall flags it honestly as a guess; it's a plausible practitioner prior, but nothing in the essay measures it, and the fraction surely varies wildly between, say, infrastructure tooling and consumer product work.

What is non-obvious is the fidelity framing. The usual critique of ticket-driven agents is that they misunderstand ambiguous instructions; Randall's point is subtler and worse — perfect understanding of an ambiguous instruction is precisely the failure, because ambiguity is where the human learning happens. The related observation that delegating implementation traps the learning inside the agent ("you get the artefact without the learning") is the essay's quiet best idea, and it connects to why the v2 spec was unwritable in advance: the implementation *was* the research.

What is weak: the evidence is one project, and a solo game at that — a domain where "the criterion lives in a person" is almost true by definition. A CRUD-backed feature with measurable conversion impact sits closer to the "reality supplies the specification" side of his line than the essay's framing suggests, and the specifiable fraction might be higher there than his guess. He also underplays the middle of his own three response modes: the "new sub-spec" path is exactly what a well-run ticket-driven pipeline could support, and he doesn't explore what infrastructure would let agents participate in that loop rather than the ticket-to-merge one.

What is left out: any engagement with automated evaluation. He grants you can build tools to accelerate feedback "once you start to understand the failure states," but doesn't consider whether simulated play (which he himself ran — "real and simulated" games) could partially compress the judgement bottleneck, or whether taste can be delegated to a model that has watched enough playthroughs. That may be right — but it's the load-bearing assumption, and it's asserted rather than argued.

## Related

- [[AI Agents Need Clear Specs]] — Eisele argues minimal specification defers rather than eliminates cost, with the optimum at structured acceptance criteria; Randall complicates that from the other side, showing that for design work the spec minimum sits at a place no amount of upfront structure reaches, because the insights only exist after building.
- [[The AI Productivity Paradox]] — Cagan locates the bottleneck in teams accelerating a broken project model; Randall strengthens this with a mechanism: the bottleneck is human judgement of the running product, which doesn't compress no matter how fast the build step gets.
- [[Laura Tacho — Data vs Hype]] — Tacho's amplifier thesis (AI amplifies existing organisational health) is echoed and sharpened by Randall's claim that measuring tickets closed will show the tools "working brilliantly while the product gets worse" — the amplification runs through whatever metric you already optimise.
- [[Specifications as the Product]] — the topic page's premise that specs are the durable artifact is nuanced by Randall's strongest case: the most important spec in his project (AI v2) was unspecifiable in advance and could only be written as the residue of a discarded implementation.

---
*Sources: [[raw/nobody-could-have-written-the-ticket]], [[summary/nobody-could-have-written-the-ticket]]*
