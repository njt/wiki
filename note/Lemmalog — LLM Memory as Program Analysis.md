# Lemmalog — LLM Memory as Program Analysis

A vulnerability researcher who wanted agents to stop resurrecting disproven hypotheses ended up writing a Datalog engine for LLM memory. The thesis: "memory" conflates two problems — retrieval ("what past info is relevant?") and state maintenance ("what is currently true?"). Vector databases solve the first; Lemmalog experiments with solving the second by treating a vulnerability investigation as a program-analysis fixed point: LLM extracts fuzzy observations into structured facts, a Datalog engine derives and *maintains* conclusions, and when an input fact changes the affected conclusions invalidate automatically instead of the model re-deriving everything from a transcript.

---

## Key Quotes

> "This is also exactly what I wanted from an LLM during vulnerability research. If an observation changes, I don't want the model to reconstruct the entire investigation from a transcript and hopefully notice all of the consequences. I want the affected conclusions to become invalid automatically."

The whole essay in two sentences. The author's diagnosis of the shared failure mode — telling an LLM something is wrong does not make it stop believing the things that depended on it — is the sharpest statement of why prompting alone can't do truth maintenance. Once a belief has downstream consequences, you need dependency tracking, not a better prompt.

> "But cosine vibe similarity and truth are not quite the same thing."

A one-line gut-punch to vector-database memory, and the cleanest phrasing of the "similarity is not relevance" critique the wiki has tracked from multiple angles ([[Context Graphs]], [[Context Rot]], [[Context Engineering at the Frontier (Linus Lee)]]). Retrieval returns what's semantically nearby; it does not know a statement was disproven two hours later, or that five conclusions depended on it.

> "This made me realise that there are really two different problems hiding under the term 'memory'."

The core contribution. Retrieval answers "what information from the past is relevant to this question?" State maintenance answers "given everything we have learned so far, what is currently true?" Most memory systems — embedding stores, summaries, transcript replay — are built for the first and silently expected to do the second.

> "we can ask why something is true."

Provenance, which Lemmalog got almost by accident (it needed dependency tracking to make incremental retraction correct), turns out to be independently valuable: when an agent asserts "we already established this pointer is attacker-controlled," you can demand the derivation chain. If no provenance supports it, it isn't part of maintained state. It doesn't stop hallucination at extraction time, but it makes unsupported conclusions much harder to smuggle in silently.

> "The amusing part is that our parser is probabilistic, while everything after it does not necessarily have to be."

The compiler analogy — LLM as front-end, Lemmalog as IR/analysis engine, another LLM invocation as back-end — is the essay's best structural insight. Probabilistic parsing is inevitable; the deterministic part doesn't have to be re-rolled on every query.

---

## Key Themes

- **#concept** — Memory as truth maintenance, not retrieval: a fixed point over facts + rules, with retraction, provenance, and validity intervals, in explicit contrast to vector similarity
- **#tool** — Lemmalog, a Datalog engine for agents, exposed via an MCP server
- **#pattern** — LLM-as-front-end / deterministic-engine-as-back-end: the fuzzy parse happens once, the consequences are computed and maintained deterministically
- **#pattern** — Incremental evaluation over re-derivation: update only the affected conclusions when an input fact changes
- **#concept** — Two-problem split under "memory": retrieval vs. current-state maintenance, which can be combined but shouldn't be conflated

## Critical Analysis

**What it gets right.** The reframe is genuinely useful and mostly missing from the rest of the memory literature. [[Agent Memory and Context]] catalogues the field's taxonomies and implementations, but nearly everything there optimises *retrieval* — what to pull back into context — while treating "what should the agent currently believe" as the model's problem to sort out live. Lemmalog is the sharpest articulation yet of why that division of labour fails: retraction, contradiction, and as-of recall are database properties, not language-model properties, and handing them to the LLM is how you get an agent confidently resurrecting a dead hypothesis two hours later.

**The benchmark honesty is the best part.** The author is explicit that Lemmalog loses to PropMem overall on both standardized benchmarks (0.463 vs 0.550 on LongMemEval; 0.533 vs 0.605 on LoCoMo), that the sample sizes are small, and that LoCoMo is conversational-memory, not vulnerability research. In a field saturated with benchmarketing, this reads as credibility. The 38× context reduction (104K → 2.7K tokens/question) is the result that actually matters for long-running agents, because extraction is paid once while full-context prompting pays the whole history on every query — the gap compounds with session length.

**The interesting number isn't the headline.** Going from 0.226 to 0.463 F1 on LongMemEval by fixing entity identity, date representation, a broken plural stemmer ("owns" never matched "own"), and a reader accidentally taught to refuse synthesis — none of it requiring a bigger model — is the essay's real thesis in evidence. The Datalog evaluator didn't get 10% smarter; the front-end and back-end plumbing did. That aligns with [[Agent Memory and Context]]'s core claim that the scaffold, not the weights, is the bottleneck.

**Where it's weak.** Inference is Lemmalog's acknowledged hole (0.164 on LoCoMo vs PropMem's 0.289), and the diagnosis is correct: flattening "I prefer quiet restaurants, except when travelling with friends" into `prefers(user, quiet_restaurants)` throws away the conditional structure before the engine sees it. The proposed fix — conditional rules and keeping episodic text around — gestures at the right architecture (deductive state + episodic memory side by side) but doesn't yet demonstrate it works. Multi-session reasoning is the other gap (0.211 vs PropMem's 0.582 on LongMemEval), and the failure there is diagnostic: information isn't mis-connected, it's never *extracted*, which is a front-end problem, not an engine problem. The author is right that this is the "important question" the essay can't yet answer: whether maintained state actually stops a long vulnerability investigation from drifting — the benchmark for that doesn't exist yet.

**What it complicates.** The essay is a useful corrective to the "just use a vector DB" default, and to the [[Memory Is a Mistake]] skepticism, both of which the hub page flags as live disagreements. Lemmalog is neither: it's a third position — structured memory is right, but the point of the structure is to *maintain truth with provenance*, not to make things retrievable. It also sharpens the distinction [[MELT]] exists to measure: Lemmalog's retractions and validity intervals are a direct implementation of MELT's correction, contradiction, and as-of recall axes, and would be a natural SUT for that harness. And against [[Zero-Mem — Zero-Token Memory Operations]], which proves you don't need *any* generated intermediate representation, Lemmalog argues the opposite direction — that generated, LLM-extracted facts plus a deductive engine buy you provenance and automatic invalidation that observed co-occurrence graphs can't.

---

*Sources: [[raw/llm-memory-program-analysis]], [[summary/llm-memory-program-analysis]]*
*Last updated: 2026-09-04*
