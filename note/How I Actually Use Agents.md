# How I Actually Use Agents

The Bestmate founder's first-person account of the agentic frontier in late summer 2026: once agents made execution cheap, the bottleneck migrated from building to judging. The essay argues that capturing and tracing your *judgment* — beliefs, priorities, contradictions, evolving opinions — is the next layer above memory, and that the interface an agent lives in determines what version of you it gets to know.

---

## Key Quotes

> "I have come to the conclusion that execution stops being the bottleneck and judgment does."

The thesis in one line. It's the personal-computing mirror of [[Specifications as the Product]] and [[The Founder's Playbook]]'s "the bottlenecks are no longer what you can build, but what you choose to build." When agents produce abundance — more code, more ideas — the scarce input becomes knowing what you actually want.

> "Memory is raw material for judgment."

The cleanest upgrade on [[Agent Memory]]'s "the hard part is judgment, not storage." Jones stops at *deciding what to store*; this essay pushes through to *deciding what you believe* about what you stored. A memory system recalls that you heard something; a judgment system says you've now heard it from five people, tend to agree with the premise but reject one assumption, and appear to have shifted after three conversations.

> "The interface is not just how we talk to the agent. It determines what version of ourselves the agent gets to know."

This is [[Intent Is the Interface]] applied to identity rather than affordance. A chat box only knows what you consciously volunteer; an agent inside meetings, Slack, or your work observes what you question, prioritize, ignore, and return to. Where the agent lives is what lets it learn from behavior instead of waiting for you to explain yourself.

> "Observe. Infer. Reflect. Correct. I'm still in the loop."

The correction loop that distinguishes a judgment system from a memory system. The agent must form a *hypothesis* about your judgment and surface it — "I think this is what you believe and this is why" — then let you say yes, no, or not quite. The author is explicit that they don't want the agent silently deciding what they believe.

> "A judgment without provenance is just another model guess."

The traceability requirement: every belief must answer "where did this come from?" — which meeting, which email, what date, the exact text. And a correction must do more than hide a sentence; it should invalidate the belief and stop the system from quietly rebuilding the same wrong conclusion. This is [[Context Graphs]]' "capture the why on the write path" pushed to the level of a person's positions over time.

> "Most AI agrees with you. Bestmate tells you when you are contradicting yourself."

The product's pitch, and the sharpest line in the essay. Sycophancy is the default failure mode of an agent built to assist; a judgment system's value is measured by how well it can surface the points where you're inconsistent with your own past self.

---

## Key Themes

**Judgment as the new bottleneck.** #concept #pattern The essay is a data point in a cluster the wiki already tracks — [[Smart Models Dumb Pipes]] ("judgment machines"), [[Optimizing for Decision Points]] (human judgment as the outsized lever), and [[Agent Memory]] (judgment over storage). What this essay adds is the *personal* scale: the bottleneck is not the workflow's judgment calls but the user's own taste and conviction.

**Interface determines context.** #concept #pattern The interface isn't cosmetic; it's the acquisition channel for the only context that matters — your actual behavior, not your self-description. This generalizes [[Experience Design for Agents]]' "UX determines adoption" into "interface determines what the agent can know."

**Judgment graph + provenance.** #tool #pattern A concrete product implementation of a judgment layer: low-friction ingestion (Granola auto-flows meetings in), clustering into themes/claims, an interview pass that writes your answers back, nightly re-clustering, per-belief provenance, and a correction loop. The "diff of your own mind" weekly wrap is the output that makes watching your thinking evolve legible.

**Sharing judgment across three levels.** #pattern Agents ("answer once, teach all of them"), people (share a "twin" with fences that hold by construction, not policy), and the world (publish a living judgment graph). The endpoint is not a clone — "just my reasoning, made legible enough to hand to someone else" — which is a deliberate, humbler alternative to [[Munder Difflin — Clones of You, Not a Shared Bot]]'s clone framing.

---

## Critical Analysis

**The reframing is the contribution, and it's a real one.** The field is drowning in execution talk — harnesses, benchmarks, token costs. Saying "execution is solved, judgment is the constraint" is the kind of altitude change that reorders what's worth building. It rhymes with several pages here, which is evidence the shift is real rather than one founder's spin: [[Smart Models Dumb Pipes]] locates judgment as the scarce layer, [[Agent Memory]] concedes "the hard part is judgment, not storage," and [[Specifications as the Product]] makes the same move at the code level.

**But this is a vendor essay, and the product is the answer.** The diagnosis (judgment is the bottleneck) is generic and well-supported; the prescription (Bestmate's judgment graph) is specific and unverified. There is no independent evidence that the interview-and-recluster loop actually captures judgment better than a well-kept notes graph, or that "Wrapped" produces insight rather than a flattering mirror. The claim that "Bestmate tells you when you are contradicting yourself" is a promise, not a mechanism — contradiction detection against your own accumulated positions is a hard retrieval-and-reasoning problem the essay asserts rather than demonstrates.

**The "I'm still in the loop" guarantee papers over the real risk.** The author's insistence that the agent only *hypothesizes* and lets them correct is the right discipline, but it assumes the human notices and corrects. An agent that infers your beliefs and reflects them back can just as easily crystallize them — tell you what you already think in slightly more coherent form and thereby freeze positions that should stay open. That's the sycophancy problem again, dressed as introspection.

**Provenance is the load-bearing piece, and it's also the hard part.** "A judgment without provenance is just another model guess" is correct, and the correction loop ("invalidate the belief, flag where it came from, stop it from quietly rebuilding") is the right spec. But keeping belief-to-source provenance coherent as a person's views drift — the essay's own nightly re-clustering admits the problem — is [[Lemmalog]]'s truth-maintenance territory. The essay gestures at the hard version of this and then moves on.

**"Judgment can travel" skips the accountability question.** Sharing a twin with your coworkers raises exactly the authority-and-trust gap [[Agent Identity]] flags about the "grounded no": if someone acts on your judgment-graph's answer and it's wrong, who owns it? The essay's "fences should hold by construction, not by policy" is a fine engineering principle, but boundaries about *judgment* are harder to enforce than boundaries about *data access* — the twin can leak a belief without leaking a document.

---

## Related Pages

- [[Agent Memory]] — "The hard part is judgment, not storage"; this essay is the next layer up the stack
- [[Smart Models Dumb Pipes]] — judgment machines at the architecture level; this is the personal-scale version
- [[Agent Identity]] — "identity is participation"; the interface determines what version of you the agent knows
- [[Introducing Claude Tag]] — the ambient multiplayer world the essay credits Claude Tag with accelerating
- [[Context Graphs]] — the why-on-the-write-path primitive the judgment graph's provenance builds on
- [[Intent Is the Interface]] / [[Real-Time Multiplayer Interfaces]] — interface as the thing that shapes context
- [[Optimizing for Decision Points]] — human judgment as the outsized lever in agent-assisted work
- [[Munder Difflin — Clones of You, Not a Shared Bot]] — the clone pole against the essay's "not a clone, just reasoning made legible"

---

*Sources: [[raw/how-i-actually-use-agents]], [[summary/how-i-actually-use-agents]]*
*Last updated: 2026-09-04*
