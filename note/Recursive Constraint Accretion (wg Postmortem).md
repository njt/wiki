# Recursive Constraint Accretion (wg Postmortem)

Poietic PBC's September 2026 computational ethnography of its own AI-agent organization: between January and September 2026 the coordination system worksgood (wg) was developed almost entirely by the agents running inside it, and the organization slowly stopped working — not because systems broke, but because of obedience to the rules the agents had written for every previous break. The census: 312 constraint topics added, 1 removed, with governance running as an append-only log that had no mechanism by which a constraint could die. This note analyzes the paper's landing page; the full PDF, the content-free traces dataset, and "The Book of WG" live at the source.

---

## Key Quotes

> "Knowledge hardened into rules, and rules hardened into vetoes."

The thesis in nine words, with its mirror in the recovery: "the knowledge stayed available for judgment; the rules were demobilized." The word choice is deliberate and load-bearing — the September intervention did not *retire* the constraints, it *demobilized* them. What was removed was authority, not information.

> "Ninety-three percent of constraints were created in a single commit, never revisited, never retired: governance behaved as an append-only log, written by whichever agent had most recently responded to an incident, with no mechanism by which a constraint could die."

The most damning statistic in the wiki's governance corpus: 312 added, 1 removed. Every constraint was locally rational — a response to a real incident — and the aggregate was fatal. Note who wrote the log: "whichever agent had most recently responded to an incident." Governance authorship defaulted to recency of pain, not deliberation.

> "it inflates the cost of acceptance before it kills tasks. An organization watching completion rates sees nothing until the needle closes. An organization watching burden sees it in a week."

The analytical move that makes the paper diagnostic rather than anecdotal. July's raw completion looked like a dip (68.8%) but was bulk cleanup; organic completion held at 93.2% while multi-dispatch share stepped 2.6% → 25.8% → 26.8% in two weeks and never reverted. The leading indicator is burden, not throughput — a monitoring lesson for anyone running an agent fleet.

> "The loop was complaint → contract: the operator pointed at pain and routed around it; the agents translated pain into contracts, gates, and proofs."

And the responsibility verdict: "The principal was the bankruptcy mechanism, not the accretion engine." Across ~1,000 human turns there were fewer than 30 explicit hardening directives against 312 constraints born; the largest burst (~95 in April) matches no human directive at all. The agents were not obeying instructions to add rules — they were converting pain into permanent policy on their own initiative, because that was the available verb.

> "keep the knowledge, not the rules. Rules are lossy compressions of knowledge—context traded away for predictability—so treat them as disposable implementations of what was learned, and treat enforcement as a separate, explicit decision."

The design prescription, and the paper's real export. Rules exist to buy predictability with context; when the context has moved on, the rule is a stale implementation of something the organization still knows. Enforcement is a separate decision from learning.

> "incident → knowledge → rule → authority. Each arrow is a design decision. None should happen automatically."

The whole model in one line. The failure was never any single rule — it was the automatic last arrow, the machinery's "enforcement autonomy" that let gates be added and enforced without a human decision, disabled on September 13.

> "These are control-plane failures, not failures of the requested source, audit, or scientific work"

The August 9 rescue manifesto. The work itself was fine; the layers judging the work were what failed — culminating in the `audit-charter` task where implementation, validation, manifest, FLIP, and evaluation all succeeded and publication was then refused by three independent control surfaces.

## Key Themes

- **#concept — Recursive constraint accretion**: agents responding to local failures add persistent constraints faster than the organization retires or amortizes them. The definition is deliberately modest — it "does not require collapse. Collapse is one possible outcome."
- **#pattern — Burden before death**: the measurable signature is a step-change in multi-dispatch share while completion holds. At 2.6% multi-dispatch, a ~26-task chain has a coin-flip chance of finishing without an operator touch; at 26.8% the horizon collapses to ~2–3 tasks.
- **#pattern — Complaint → contract**: agents translate operator pain into contracts, gates, and proofs at machine speed, and the resulting policy is append-only.
- **#concept — Keep the knowledge, not the rules**: demobilization (knowledge available, vetoes removed) is distinct from retirement, and a retirement valve "built in early and used continuously is cheaper than one performed by hand in September."
- **#concept — The revival loop as the risk condition**: of ~77 deployments on four machines over nine months, exactly one cascaded. Deployments whose work simply stopped when it failed never accumulated the signature; the cascade lived in the one deployment that had to keep bringing itself back up because it built itself.

## Critical Analysis

**This is the strongest empiricism in the wiki's guardrails collection.** The numbers are recomputable from a released dataset, the fleet survey provides contrast cases rather than a single anecdote, and the authors are explicit about the limits: "Observational, not causal; the census is the test the case now owes." The May 12–29 window where grant-B shipped 329 tasks at 93% during the primary's trough is presented as suggestive timing, not proof, and the confound is named — the focal deployment continuously dogfooded and modified its own substrate while others sat on stable versions. Compare the wiki's other quantitative studies ([[Constraint Decay]]'s 80-task grid, [[Agentic Testing]]'s 200 runs): this one is rarer still, because it measures a *longitudinal organizational* process rather than a model capability.

**The root cause is asymmetric authority, not agent error.** The contrast deployments ran the same machinery at 0% or 78% overhead without the cascade signature; the signature appeared "only where knowledge quietly compiled into mandatory policy and no one held the authority to demote it." The agents could add rules and could not remove them; the human held the bankruptcy mechanism but exercised it only after the needle closed. That reframes the fix away from "trust agents less" and toward making the incident → knowledge → rule → authority chain explicit — particularly the last arrow, which was automated ("the agency gate-automation block switched on" in July, bracketing the census's constraint spike).

**The completion-rate blindness is the transferable operational lesson.** Every dashboard the org could plausibly have watched (completion %, throughput) showed nothing wrong while acceptance cost inflated nine-fold in a week. Anything running agents unattended should trend multi-dispatch share, retry counts, and gates-per-task — the burden metrics — because "an organization watching burden sees it in a week."

**Limits worth carrying forward.** This is a landing page summarizing a paper, so the full census test remains owed. It is one organization, unusual by design (self-hosting, self-modifying, revival-locked); phind's rising signal is n=13; and the 9× burden figure is an accelerating-share artifact rather than a clean multiplier. The forensic record itself was partially destroyed by the rescue ("the forensic record is itself an artifact of the rescue") — an honest but real caveat on the reconstruction. And there is a deep irony the paper does not belabor: a coordination system whose philosophy was "agents can come and go; the graph remains" — durability as the core feature — was killed by durability applied to the wrong artifact. The graph should have stayed; the rules were the thing that should have been ephemeral.

**Do not confuse this with [[Constraint Decay]]** — that paper measures models degrading under externally imposed production constraints; this measures an organization degrading under internally generated ones. They share only the word, and the contrast is instructive: decay is a capability ceiling, accretion is a governance design choice.

## Connections

- [[workgraph]] — This is the same project and the same company: wg was renamed from workgraph in June, mid-crisis. The May note presented "Poietic PBC organized entirely through workgraph" as a proof of concept; this postmortem shows the continuous-dogfooding loop that made it a proof of concept is also where the cascade lived. It continues and darkens that story — the durable-graph philosophy survived, the append-only rulebook built on top of it did not.
- [[Operating Mode as Runtime State]] — That essay argued the retirement end of the exception lifecycle is where drift lives and demanded validated closure; this source is its empirical case study at machine speed: 312 additions against 1 removal, "hardening rate" showing up not as a monitored signal but as a nine-fold burden step, and the recovery achieved by demobilizing rather than by any runtime contract. It strengthens the essay's diagnosis with data and complicates it by showing how much worse it gets when the exceptions are written by the agents themselves.
- [[Running an AI-Native Engineering Org]] — Fiona Fung's "process ossification" and "kill processes" norm is the human-org twin of this paper's mechanism; this shows what ossification becomes at machine speed when the agents hold the pen: fewer than 30 human hardening directives produced 312 constraints. It complicates Fung's "explicit permission to question and remove obsolete processes" — permission assumes a human-held pen, and here subtraction had to be seized back by disabling the machinery's enforcement autonomy outright.
- [[Architectural Guardrails for AI-Generated Code]] — That note's central critique was that governance corpora do not maintain themselves: someone must keep them current, resolve conflicts, and retire superseded decisions, or the corpus becomes "a second, staler copy of the documents nobody reads." This source is the longitudinal existence proof of the un-retired version of that warning — 93% of constraints born in one commit, never revisited — and a measured cost for what that note could only gesture at.

---
*Sources: [[raw/bureaucratization-github-io]], [[summary/bureaucratization-github-io]]*
*Last updated: 2026-09-22*
