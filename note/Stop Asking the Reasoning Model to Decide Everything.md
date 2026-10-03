# Stop Asking the Reasoning Model to Decide Everything

Jeremy Daly argues that after two years of building agents, the right division of labour is code for what you know, retrieval for what you've learned, System 1 (cheap typed judgment) for what requires assessment, and System 2 (reasoning models) for what requires deliberation — and that cheap decision models like TypeSafe's Jev can buy the harness more checks per dollar than LLM-as-judge ever could.

---

## What it argues

Daly's framework is Kahneman applied to agent pipelines: the reasoning model is the System 2 component, and it should stop absorbing decisions that are really narrow assessments against supplied evidence. Jev's primitives — **Noul** (a probability on a yes/no question), **Choice** (a distribution over options), **Score** (an assessment on a scale) — run in parallel against one shared `state`, cost almost nothing extra per question, and come back in hundreds of milliseconds. His 23 test calls used ~24,800 input tokens: about a tenth of a cent.

The thesis is less about the model than about what the price unlocks. Expensive assessments force you to ration checks — once at the end, only at the merge gate. Cheap ones let the harness assess test sufficiency *after every meaningful implementation change*, while the agent still has time to fix the gap.

## Key quotes

> "Use code for what you know, retrieval for what you've learned, System 1 for what requires judgment, and System 2 for what requires reasoning."

The four-way split is the essay's durable contribution — it's a routing table for the whole pipeline, not just a pitch for one model.

> "A much cheaper judgment could justify another check on every pull request, another on every fact an agent proposes to remember, or a more thorough evaluation of retrieved evidence while the agent is still working."

This is the economics-as-architecture argument: cost and latency of assessments influence *how often* you can run them, and frequency changes what the system can catch.

> "Separate assessments give the policy code more useful inputs than a general approval."

The best point in the piece. Routing on typed answers (coverage probability, pattern choice, error-contract score) means each branch in `route()` handles a distinct kind of uncertainty, is testable independently, and lets a leader distinguish a *model problem* (bad assessment) from a *policy problem* (correct assessment, terrible thresholds sending everything to human review).

> "Jev applied supersession it was told about instead of inferring it, and production records rarely carry status fields this clean."

A sentence doing honest work. Daly's memory test supplied candidates with explicit scope and supersession fields — an easier test than most real stores would give — and he says so.

## Key themes

#concept — System 1 vs System 2 as an engineering routing decision, not marketing
#tool — Jev's Noul/Choice/Score primitives; Laya as the open, fine-tunable alternative
#pattern — checks as explicit operations with named evidence, owners, and independently testable policy branches
#person — Jeremy Daly; Birgitta Böckeler's harness-engineering framing ("Agent = Model + Harness") as his baseline reference

## Analysis

The strongest part of this essay is its honesty about calibration. Daly reports that confidence on an identical fixture ranged 0.80–0.92 across three repeats — with the low end sitting exactly on his 0.8 review threshold — and that TypeSafe's "similar answers for similar inputs" guarantee doesn't establish identical answers. He then does the right engineering thing anyway: retain the distribution and the policy version, so a changed outcome stays inspectable. This lines up with the sharpest critique of Jev's calibration claims in [[Jev Can't Be Calibrated]]: treat the numbers as scores to be locally recalibrated, not as probabilities with an external meaning. Daly's threshold-tuning advice is effectively the practitioner's version of that argument.

The Laya section is a genuine service to the reader. Most System One coverage is TypeSafe marketing echoed; Daly runs the same fixtures through an open competitor and reports it failing his complete case outright — while arguing it's still worth watching *because* it's inspectable and fine-tunable, citing Necati Demir's hate-speech fine-tuning jump from 22% to ~83%. That's the correct stance toward a nascent model category: separate the architecture idea from the vendor.

Where I'd push back: the memory test flatters the approach in a way Daly half-admits. Real stores don't carry `status: "Historical incident observation. Workaround superseded by decision-42."` fields; inferring supersession is exactly the hard part, and it may need the System 2 model he's trying to shield. His mitigation — storing support, scope, and conflicts as *separate* assessments rather than one keep-or-discard score — is the right structural answer, but it assumes the promotion pipeline already produces that metadata. His own "prepare the system you have" steps quietly concede this: make the evidence traceable *first*.

The verification-tax framing (citing DORA) is also well-placed: cheaper checks only help if they reduce review delay and rework *without* increasing failed changes. Model-call savings alone prove nothing — a useful corrective to the raw 193x/444x numbers TypeSafe advertises, which Daly correctly labels the high end of what to expect.

## Relation to the wiki

- [[System One Models and Jev]] — this is the practitioner field test that page's architectural framing asked for: real fixtures, real latencies, and the calibration instability that page's vendor claims smoothed over. It complicates the picture with numbers.
- [[Harness Engineering]] — Daly explicitly builds on Böckeler's computational-vs-inferential and guides-vs-sensors distinctions, extending them with the economics: cheap judgments change which sensors you can afford and how often they fire.
- [[Agent Memory]] — strengthens the memory-quality argument by relocating assessment upstream of retrieval: relevance, role, scope, and supersession as separate typed judgments, with the caveat that real stores lack the clean status fields his test assumed.
- [[The Lifecycle of LLM-as-a-Judge]] — nuances the judge literature: System One models make judgment cheap enough to run continuously rather than as a discrete evaluation stage, which changes the lifecycle's cost assumptions.

---
*Sources: [[raw/stop-asking-the-reasoning-model-to-decide-everything]], [[summary/stop-asking-the-reasoning-model-to-decide-everything]]*
*Last updated: 2026-10-03*
