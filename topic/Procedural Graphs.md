# Procedural Graphs

An arXiv paper proposing the Procedural Graph (PG), an explicit, editable directed graph that stores an LLM agent's *procedural* knowledge — how to act, in what order, under which conditions — as (procedure, relation, procedure) triplets, mirroring how knowledge graphs organize factual knowledge. Nodes abstract tool calls, reasoning steps, and states; edges encode admissible transitions annotated with *condition*, *guidance*, and *pitfalls*. A guidance model localizes the agent's active node and verbalizes the surrounding subgraph into step-level situational guidance, while an offline self-evolution loop contrasts failed and successful trajectories to propose graph edits, gated by held-out validation and a rejection memory.

---

## The Core Idea

The motivating problem is that most agents select actions through unconstrained generation over a flat, growing log of prior actions and observations. As trajectories lengthen, the burden of procedural coherence falls entirely on free-form generation, and the documented failure modes follow: losing track of objectives, invoking tools out of order, repeating unproductive actions.

The Procedural Graph's answer is to lift procedural knowledge out of the model weights and the log, into an inspectable, editable, retrievable structure. The analogy is explicit and sharp:

> "just as a knowledge graph organizes factual knowledge into (entity, relation, entity) triplets for *what-is* questions, a Procedural Graph organizes procedural knowledge into (procedure, relation, procedure) triplets for *what-to-do* questions."

The two-phase design separates use from improvement. Online, the graph is frozen: locate the active node from the trajectory, extract its neighborhood, and have a guidance model generate situational advice. Offline, a refiner mutates the graph's topology and attributes against execution feedback. This split — frozen at inference, mutable in training — is the same discipline the [[Harness Engineering for Self-Improvement]] lineage applies to harness code.

## Key Quotes

> "The remaining challenge is to combine an editable procedure representation with guidance that is conditioned on the agent's current progress."

This is the paper's honest diagnosis of the gap it fills. Memory and reflection methods (Reflexion, ExpeL) record experience as text but leave the solver to reconstruct *how* it applies now. Workflows and state machines make steps explicit but need manual design. PG wants both: editable structure plus step-local guidance.

> "The graph keeps a task domain's procedural knowledge outside the model weights, where it can be inspected, retrieved at each step, and edited without retraining."

The editability is the whole point. This is procedural memory as infrastructure — a direct implementation of the quadrant [[Agent Memory]]'s taxonomy names as "the most under-exploited type," and of CoALA's procedural-memory module, which the paper itself cites.

> "Retrieving the connected neighborhood exposes both the action and its procedural prerequisites."

The argument against naive per-transition retrieval: top-k similarity over isolated attributes can omit the *verification* step that makes a later action appropriate. Localized subgraph retrieval — the action plus its topological context — is the design move that distinguishes PG from [[Context Graphs]]' typed-edge memory and from AutoGuide's state-conditioned guideline retrieval.

> "Rejected candidates are retained as negative constraints."

The rejection memory (Step 4 of the evolution loop) is the most interesting engineering detail. Iterative self-correction tends to re-propose the same failed edit; logging rejected graphs and feeding them back to the refiner as negative evidence is a cheap, structural guard against looping. Compare the mutation-and-gate pattern in [[Harness Engineering for Self-Improvement]]'s AFlow and Self-Harness lineage.

> "What changes under guidance is *which* tools are called and when, rather than simply how many."

The EnterpriseArena result in one line. On some models the evolved graph *reduces* tool calls per month, on others it *increases* them — the win isn't efficiency per se but sequencing: running forecast and market checks before a financing decision, and requesting capital months early because delivery lags one to six months. Anticipatory fundraising is the behavior that tracks survival across all four models.

## Key Themes

- **#concept** — Procedural memory as a graph of attributed transitions, the CoALA quadrant the rest of the field leaves implicit in weights or scattered across prompt templates
- **#pattern** — Localized subgraph guidance: locate the active node, extract the neighborhood, generate situational advice — steering without dictating
- **#pattern** — Validation-gated self-evolution: refiner proposes edits, held-out gate accepts/rejects, rejection memory suppresses repeats
- **#tool** — The relation vocabulary (LEADS_TO, TRIGGERS, PROVIDES_INPUT_FOR, CONVERGES_TO) and the condition/guidance/pitfalls edge attributes

## Critical Analysis

**The strongest result is the loop repairing a flawed expert prior, not the headline win rate.** The MultiChallenge construction study is the paper's most persuasive evidence: a hand-crafted expert graph *lowered* success from 87.5 to 58.9, a single static update made it worse (53.6), and iterative evolution recovered to 92.9. That a prior which actively hurts can be rescued — not just built-from-scratch — is the result that makes self-evolution a deployment strategy rather than a benchmark trick. Most self-improvement papers only show scratch-to-good; few show bad-prior-to-good.

**The evaluation is unusually heavy on long-horizon realism.** EnterpriseArena — a 132-month CFO simulation with undisclosed crises, delivery lags, and market caps — is a far better stress test than the typical benchmark suite, and the round-by-round evolution trace (Rounds 1–10, with rollbacks and a structural-failure skip) is refreshingly transparent about how noisy accept/reject decisions are ("individual accept/reject decisions turn on one or two episodes"). The authors explicitly report the returned graph's test score rather than the best single round, declining to select on the test set. That honesty is rarer than it should be.

**The efficiency tax is real and under-theorized.** Guidance adds an extra LLM call per step, and total token use rises even when it shortens trajectories. The paper's own future-work note — reuse guidance across steps or generate it selectively — concedes the obvious objection. Localized subgraph guidance is cheaper than full-graph generative guidance, but still more expensive than no graph. For a method pitched partly as reducing redundant tool calls, the token overhead is the clearest gap to close.

**The author list is absent from the fetched HTML**, so provenance is thin — a reminder that arXiv HTML extraction can drop metadata. The models evaluated (Claude Sonnet 4.6, Gemini 3.1 Pro, Grok 4.1 Fast) date the paper to 2026, but there's no named author to hold to account.

**Where it sits in the wiki.** PG is the *procedural* counterpart to [[Context Graphs]]' *decision* graphs — the former encodes what-to-do transitions, the latter encodes why-decisions. Both reject "similarity is not relevance" as their shared starting point, but PG's answer is topological neighborhood retrieval rather than typed edges over facts. It is a concrete instance of the self-improving-harness pattern [[Harness Engineering for Self-Improvement]] surveys (AFlow's workflow-as-graph-search is a direct ancestor), and a working implementation of the procedural-memory quadrant [[Agent Memory]] names and [[How AI Agent Memory Works]] maps to tool invocation.

---

*Sources: [[raw/2609-09153v1]], [[summary/2609-09153v1]]*
*Last updated: 2026-09-11*
