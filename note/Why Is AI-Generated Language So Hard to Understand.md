# Why Is AI-Generated Language So Hard to Understand?

A psycholinguistics paper that does the rare thing: instead of cataloguing AI prose tics anecdotally, it measures known comprehension-cost features (object relatives, dependency length, stacked nouns, denominal verbs, inanimate subjects, rare n-grams) across 14 models and three settings, then causally tests the hypothesis that verifiable-reward post-training pushes models toward information density at the expense of human intelligibility.

---

## Precis

Human language is shaped by a pressure toward communicative success under cognitive constraints — lossy working memory, incremental processing, dependency-length minimization, uniform information density. LLMs inherit those signatures from pre-training on human text, but modern post-training (RLVR for math, code, reasoning) rewards correct answers, not clarity, and long context windows mean the model never pays the memory cost that forces humans to write short dependencies. The paper finds frontier models systematically overuse constructions that are grammatical but out-of-distribution for human language, and demonstrates a length penalty alone is enough to produce categorical compression preferences that transfer to unseen constructions.

## Key quotes

> "Your instinct is right; the convoluted sentence you are currently reading is quietly eating its own reader-patience budget — footnote it."

The paper's opening joke is also its specimen: an object relative, a denominal verb, an inanimate subject, and a stacked noun compound in one sentence. Every feature is one that saves tokens while costing the reader processing effort.

> "The use of verifiable rewards creates an environment where correctness is decoupled from humanlike communication, leading models to converge toward a novel, potentially less comprehensible distribution of language use."

This is the mechanism claim, and it's the sharpest idea in the paper: RLHF accidentally rewarded verbosity (humans like fuller answers), so the shift to RLVR didn't just remove bias — it removed the last remaining pressure for communicative success.

> "The models were still able to succeed at their task, since both members of an alternation were valid solutions."

Crucial detail: the compression never hurt task reward. The shorter variants were always valid. Nothing in the training signal could distinguish "dense but hard to read" from "dense and fine," so the model had no reason to care — the misalignment is invisible to the objective.

## Key themes

#concept #ai-research #psycholinguistics

## Analysis

The round-trip translation design is what elevates this above the usual "LLMs write weird" genre. By holding semantics fixed and observing which features the model chose over available alternatives, it converts stylistic anecdote into a measurement of preference — and the fact that the effects grow from base → instruct → Think within the Olmo family is a natural experiment in the dose-response relationship between reasoning post-training and linguistic drift.

The stakes go beyond style. Agent-written code review summaries, PR descriptions, and handoff documents are now the interface between agents and the humans supervising them; text that systematically increases processing cost taxes exactly the review-and-monitor function that agentic workflows depend on. The paper's own remedy — reward effective communication — is reasonable, but its data suggests the deeper problem: an objective that never measures the reader will never select for the reader.

## Related pages

This paper gives the quantitative, causal backbone for [[Why Does AI Write Like That]] — Sam Kriss's overfitting story explains the *stylistic* tics; this paper shows the *processing-cost* features are a measurable, post-training-driven drift, and that reasoning training (Olmo Think) intensifies it. It sharpens [[What Broke and Why — RL Post-Training]]'s practitioner account of RL post-training: the length-penalty experiment shows that even a mundane token-efficiency term, with no intent to change style, is sufficient to reshape grammatical preference. And it puts numbers under [[Slop Score]], which measures the same over-represented patterns from the outside; this paper explains *why* those patterns exist — compression pressure, not sloppy imitation.

---
*Sources: [[raw/2610-29162v1]], [[summary/2610-29162v1]]*
*Last updated: 2026-11-02*
