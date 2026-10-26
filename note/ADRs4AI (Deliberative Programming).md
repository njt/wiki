# ADRs4AI (Deliberative Programming)

ADRs4AI is a method (and loose toolkit: template repo, editor extension, MCP server) that makes the human-AI thinking process itself the durable artifact: conversations are chunked into git-tracked ADRs, each open question is staked as a `### QST:` the AI answers with options, recommendation, rationale, confidence and a falsifier, and the human rules in one line. The pitch is that the most valuable byproduct of collaboration with an AI is the thinking, and it normally evaporates; the record keeps the why so decisions compound and work is resumable.

---

## The mechanic

One seed, one question, one ruling. You dump messy, dictated thinking into a seed; the AI chunks it into ADRs that keep your actual words, the options, the reasoning, and the open questions. Each staked question arrives with the AI's options and recommendation — so answering streamlines to accept or override. Every open question is findable by one grep:

```
grep -rn '^### QST' docs/adr/
```

The showcase example, verbatim from the project's own record, is the whole thesis in miniature: GPT 5.6 Sol supplied a full recommendation (options, rationale, confidence, falsifier) on a bitemporal question; the founder ruled "C — Bitemporal hybrid" from his phone. The AI carries the reasoning; the human spends one judgment; the record keeps the why.

What it explicitly refuses to be: transcript archival, vector-database memory, an orchestration platform, or a claim of AI persistent identity. "The record holds, minds change, and the work resumes either way."

## Key quotes

> The most valuable byproduct of the collaboration between a human and an AI is the thinking — and in ordinary practice, all of it evaporates.

This is the founding claim, and it's a sharper framing than the usual "context engineering" pitch: the artifact worth saving is not the code or the prompt but the deliberation that produced them.

> **That's the whole mechanic**: The AI carries the reasoning, the human spends one judgment — accept, override, or any answer in any shape — and the record keeps the why, for every mind that comes later.

Note "any answer in any shape" — the method does not force the human through the AI's option lattice. That flexibility is what separates it from approval-gate review workflows.

> It does not make an AI persistent, and it does not make anyone remember. It makes the work resumable — and the thinking compound.

A deliberate anti-marketing distinction, and a correct one: memory architectures chase persistence of the agent; this chases persistence of the *reasoning*, which survives model changes and crew turnover.

## Themes

#concept — deliberative programming: decisions as the unit of human-AI collaboration
#tool — ADRs4AI template, editor extension, MCP server
#pattern — staked questions with falsifiers; accept-or-oversee rulings; git as the substrate

## Analysis

The genuinely interesting move here is making the falsifier a first-class field. An AI recommendation that states what would prove it wrong is a very different object from an AI recommendation — it converts the human's accept/override from a trust decision into a checkable bet, and it gives future agents (and humans) the exact conditions under which to reopen the ruling. That's a stronger epistemic contract than most "decision record" practices, which record the conclusion and let the reasoning rot.

The friction is predictably human. The demo shows the loop at its best — one well-formed question, one crisp ruling. Real projects will produce dozens of questions a day, and the accept-or-override streamlining that reduces cognitive load at n=16 questions (the extension screenshot) may become rubber-stamping at n=75. The method's own emphasis on "the record begins before the building does" is the right discipline, but nothing shown enforces that the human actually read the rationale before ruling — the same rubber-stamp collapse that killed mandatory human code review looms here, just moved to the decision layer. The mitigations are structural: questions are grep-able, supersession is modeled (the bitemporal ruling is the method eating its own dog food), and the record is plain git markdown, so worst case it degrades to ordinary ADRs. That's a low floor. The open question is whether one-line rulings preserve enough of the *human's* why — the record is heavily attributed on the AI side, and a founder's "C" from a phone is a thin trace of a judgment. As a complement to how agents handle stale or over-litigated decisions in practice, this lands firmly on the optimistic side of the [[Coding Agents Love Decision Records]] debate: conventions and structure in, unreadable prose out.

## Related

- Strengthens [[Coding Agents Love Decision Records]]: that source diagnoses agents over-adhering to stale ADRs and over-litigating updates; ADRs4AI answers with a concrete mechanism — staked questions with falsifiers and explicit supersession — that makes reopening a ruling a first-class act rather than prose churn.
- Extends [[IDRs as Records of Intent]] toward an operational protocol: where IDR capture is about recording intent after the fact, ADRs4AI puts the question *before* the work ("the record begins before the building does") and makes answering the primary human act.
- Complicates [[Maybe Coding Agents Don't Need a Bigger Memory]] by agreeing with its premise from the opposite direction: the point is not agent-side memory but a shared, greppable record both minds read — "the record does the remembering," not the model.
- Nuances [[Capturing Why Engineering Decisions]] with the falsifier field and explicit confidence: recording not just why a decision was made but what would overturn it, which is what makes the record auditable rather than merely archival.

---
*Sources: [[raw/adrs-systems]], [[summary/adrs-systems]]*
*Last updated: 2026-10-26*
