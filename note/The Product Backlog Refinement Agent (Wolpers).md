# The Product Backlog Refinement Agent (Wolpers)

Stefan Wolpers designs the boundary for an AI agent that prepares Product Backlog refinement, arguing the real risk is not the agent doing the team's homework but the agent settling the team's questions in advance: a brief that "suggests" closing, splitting, or reordering items converts preparation into decision before the team ever meets. The remedy is a written boundary — mark, suggest, never solve — enforced through stop rules and verified by a Product Owner who routes every finding.

---

## What the source argues

An agent asked to "help with refinement" can capture the conversation without writing a single acceptance criterion. The obvious failure — an agent drafting backlog items the team rubber-stamps — is visible and therefore easy to catch. The hard failure looks like help: a brief recommending that one item be closed as a duplicate, another split, a third created, and four items flagged "ready candidates" reads as friendly advice while every line pre-answers a question the team was supposed to work out.

Wolpers' remedy has four parts:

- **Four evidence checks** the agent may run against the team's own records (Product Goal, roadmap change log, anti-product backlog, Sprint Review notes, Retrospective actions): readiness, goal fit (including detecting *prioritization by proxy* — a stakeholder pushing an item to the top through the Product Owner), outdated items, and missing work.
- **A four-part test** separating suggestion from decision: rests on supplied records, makes uncertainty visible, leaves materially different answers open, does not assume the disputed resolution.
- **Written stop rules** in an A3 Handoff Canvas before the first run: never write acceptance criteria, never estimate, never split/reorder/remove items, never advise closing or creating, never propose Sprint content or goal, never invent a source. Missing records must be reported as "what I could not check," not inferred as nonexistent.
- **Human verification**: the Product Owner opens the record behind every agenda-changing finding, looks at things the brief did *not* flag (it cannot report its own omissions), and routes each finding — PO housekeeping, discovery, refinement agenda, or impediment.

The grounding for why "just a suggestion" is dangerous is empirical: a preregistered study (Williams-Ceci, Naaman et al., Science Advances, 2026; n=2,582) found biased AI writing assistants shifted writers' attitudes toward the assistant's position, mostly unnoticed, and warnings did not mitigate the shift.

## Key quotes

> "A recommendation decides nothing until someone accepts it and acts on it. Once it sits on the agenda, however, the team is likely to discuss the recommendation instead of the question behind it, and that is how preparation quietly turns into a decision."

This is the article's sharpest insight: the failure mode isn't the agent's output being wrong, it's the output *reframing the conversation*. Once the agenda carries the agent's resolution, the team debates yes/no on a pre-chosen answer instead of the open question.

> "My rule for such a refinement agent is short: mark, suggest, never solve."

A crisp autonomy boundary, notable for being stated in the agent's instruction "word for word" rather than left as culture. Compare [[The Clanker Constitution]]'s behavioural baseline for coding agents — same move, different domain.

> "Never invent a source, a reason, or a figure. (If a record needed for a check was not supplied … state that limitation under 'What I could not check.' Do not infer that the record does not exist.)"

The quiet last rule is the best one: it targets the model's tendency to convert absence of evidence into evidence of absence — reading a planned experiment as a commitment, or a hypothesis as a result.

> "A brief cannot report its own omissions."

A one-line argument for why human verification can't be a spot-check of the flagged items only; the Product Owner must also inspect what the brief never mentions.

## Themes

#concept #pattern #tool

## Analysis

This article is a well-executed application of a pattern the wiki has documented for coding agents, transplanted into product management: the agent as advisory reader, the human as decision owner, and the boundary written down before the first run. Wolpers' "mark, suggest, never solve" maps almost exactly onto the Assist tier of delegation frameworks, and the four-part suggestion test is one of the more operationalizable versions of "advisory vs. agentic" I've seen — most sources assert the distinction; this one gives you a checkable criterion.

What elevates it above a Scrum-blog piece is the anchoring evidence. Most autonomy-boundary arguments rest on intuition ("humans should stay in the loop"); Wolpers cites a 2026 study showing that even *static suggestions* shift judgment and warnings don't help — which undermines the common mitigation of "we'll just tell people the AI output is advisory." The honest implication, which he doesn't fully draw out, is that even a compliant brief shapes the Product Owner's judgment, so the stop rules protect the *team's* conversation but not fully the *PO's* reading of the evidence.

Caveats: the piece is heavily promotional (the A3 Delegation System templates and a $149 course are woven through it), and it's prescriptive rather than empirical — no field data on whether teams actually keep these boundaries under deadline pressure, which is exactly when the "human desire to avoid unnecessary work" temptation bites. The fetched copy is also truncated mid-sentence in the routing section. Still, as a design essay for agent boundaries outside the code editor, it's one of the cleaner statements of the pattern.

## Related pages

- [[Handing the Agent the Whole Job]] — Bertram's four-part limit set (budget, write paths, credentials, approvals) written before the first run is the coding-workflow twin of Wolpers' stop rules; this source strengthens its claim that boundaries must precede delegation, and extends it to non-coding knowledge work.
- [[You Shall Not Pass — Where Developers Draw the Line on AI Autonomy]] — The Microsoft L1–L5 autonomy survey gives the quantitative backdrop for Wolpers' Assist-tier placement; this source nuances it by showing the line drawn not by task risk but by *conversational ownership* — who is supposed to reach the answer.
- [[Optimizing for Decision Points]] — Simister argues workflows should surface taste-sensitive decisions rather than let models fill them with safe defaults; Wolpers' four-part suggestion test is a concrete mechanism for exactly that, applied to backlog refinement.
- [[The Clanker Constitution]] — Kenn's behavioural baseline for coding agents parallels Wolpers' word-for-word stop rules; both treat written operating principles as enforceable agent constraints rather than aspirations.

---
*Sources: [[raw/product-backlog-refinement-agent]], [[summary/product-backlog-refinement-agent]]*
*Last updated: 2026-10-26*
