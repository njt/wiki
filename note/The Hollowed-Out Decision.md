# The Hollowed-Out Decision

zero_shift's HN comment traces a single workplace decision — an AI feature in their company's product — back through its lineage and finds no origin point with a human behind it: Claude wrote the code from a prompt for an Atlassian-AI ticket, generated from AI-digested docs, derived from strategy memos almost certainly written by Claude, in service of a strategy chosen by execs who now communicate mostly via AI-written memos and justify themselves by citing "tech influencers, market conditions, customer expectations." The comment's conclusion: nobody in this ecosystem is actually in control, and the whole stack has been "hollowed out, replaced either by inscrutable machines, or inscrutable incentives."

---

## Key quotes

> The code was stamped by Claude driven by a prompt. The prompt was for a ticket generated with the Atlassian AI integration. Atlassian had digested docs made with AI. The docs came from strategy memos I'm 90% sure were written entirely by Claude.

The lineage as a chain of AI-to-AI hops with humans reduced to relay points. What makes it devastating is that each hop is individually defensible — ticketing automation, doc summarisation, memo drafting — and no single hop is the villain. The corruption is in the composition.

> We were not building the feature because we wanted it. We were building it because we thought other people expected it.

The crispest statement of the comment's thesis. Intention has been replaced by anticipated expectation — and the expectation itself is unverified, circling back to market hype.

> Nobody in this ecosystem, I thought, is actually in control here. Nobody is actually orienting work and action to real, concrete goals. It's all based on speculation and anxiety about the future.

The claim is stronger than "AI-generated work is low quality." It is that the *motivation* for work has become unexplainable — not that nobody understands what the code does, but that nobody can say why it exists.

> It has all been hollowed out, replaced either be inscrutable machines, or inscrutable incentives. Ironically it rather resembles the kind of "misaligned" superintelligence we are supposed to be avoiding.

The closing analogy does real work: the misaligned optimizer was supposed to arrive as a machine, but here it arrives as an *organisation* — a system optimising toward a reward signal (investor expectation, hype) that no participant endorses or understands.

## Key themes

#concept #culture #decision-making #hype

## Analysis

This is a comment, not an essay, and its power comes from being a first-hand trace rather than an argument. The author isn't theorising about AI in the workplace; they followed one concrete decision to its source and hit fog. That's the same method as a postmortem, applied to strategy instead of an outage — and it's the method most "AI adoption" writing avoids, because the lineage always looks like this if you actually follow it.

Two things deserve pushback. First, "nobody is in control" is partly the permanent condition of large organisations — strategy memos were ghostwritten, expectations were socially constructed, and execs cited influencers long before Claude. The AI layer accelerates and densifies this, but claiming it's *new* overstates the case, and the author's own hedge ("Arguably there has been several layers of human review") concedes as much. Second, the misaligned-superintelligence analogy is rhetorically delicious but analytically loose: there's no optimizer here, just correlated laziness — each human along the chain offloaded the writing, and no one did the thinking that writing used to force. That's a governance failure, not an alignment failure, and it has a more mundane fix: make someone own the decision.

Still, the note that "several layers of human review" can coexist with zero intention is the most uncomfortable observation in the piece. Review without intent is rubber-stamping, at every level of the org chart.

## Related pages

- [[AI Mania Is Eviscerating Global Decision-Making]] — makes the same diagnosis at geopolitical scale; this comment is the single-company, first-hand specimen of that thesis.
- [[Nobody Could Have Written the Ticket]] — the pipeline-side mirror image: both argue the ticket/spec is where intent should live and where it actually goes missing; this comment adds the exec layer above the ticket.
- [[The Flat Curve Society]] — Yegge's market-side froth and speculation maps directly onto the "inscrutable incentives" half of this comment's diagnosis.
- [[Capturing Why Engineering Decisions]] — the constructive counterpart: recording the *why* of decisions is precisely the trace this author couldn't reconstruct.

---
*Sources: [[raw/item]], [[summary/item]]*
*Last updated: 2026-09-29*
