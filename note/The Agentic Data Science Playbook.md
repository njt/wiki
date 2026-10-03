# The Agentic Data Science Playbook

Hugo Bowne-Anderson and Eric Ma demonstrate that a capable model plus a business question is not enough for agentic data work: Claude Opus 5.0, asked simply to "build a fraud detector," produced a textbook-worthy evaluation failure (random time splits, a planted leakage feature) reporting F1 0.87 that collapsed to an honest 0.70 under correct methodology. The article turns that failure into a five-practice playbook — frame, equip, organize, review independently, and convert lessons into skills and evals — arguing the data scientist's job becomes specification and verification, with the agent carrying the investigation itself.

---

## The sabotage experiment as thesis

The planted-leakage setup is the article's best move. Instead of asserting that agents need guidance, the authors engineer the failure and measure the gap:

> "Claude wrote the code, trained a random forest, and reported an F1 of 0.87 and ROC AUC of 0.99. It had split transactions randomly, mixing earlier and later time steps... Claude also used a feature we had planted as a proxy for the fraud label (yes, we tricked it!)."

This is data science's version of the sycophancy-and-verification problem in coding: the agent optimizes the metric it can see, not the one that matters. Under corrected evaluation, recall on high-degree nodes was 0.21 — the subgroup that mattered most operationally. The lesson generalizes far beyond ML: an agent that can't be told what "success" means will confidently report a success that is fiction.

## Five practices, and what's genuinely new

1. **Frame the investigation.** A brief that specifies the decision, constraints, and evidence — not the implementation. "Classify transactions from later time steps using only information available when each is scored" matters more than "use a random forest." Crucially: "specify what the investigation must establish without specifying the answer you want."
2. **Equip the agent.** Runtime, workspace layout, skills, and data documentation. A skill "specific enough to change the agent's behavior" — "Be rigorous" gives it nothing; a fraud skill mandates feature-availability checks, temporal evaluation, subgroup reporting.
3. **Organize the work.** Bounded autoresearcher loops against a frozen evaluator for predictive questions (41 overnight experiments, validation loss down ~70%, F1 0.72→0.82); parallel analyses over competing assumptions for causal questions, where no held-out outcome adjudicates.
4. **Review independently.** A fresh adversarial reviewer — but with an important subtlety: "A fresh agent session is not necessarily an independent review if it can read the investigator's earlier attempts through the workspace or Git history."
5. **Convert lessons into skills and evals.** The planted-leakage failure becomes a reusable check plus an eval case — evidence that the agent's *analytical behavior* improves, not "a growing collection of instructions that merely sound sensible."

## Key themes

#concept #pattern #tool

- **Specification and verification as the two central responsibilities** — the data science restatement of the outer-loop argument in [[Own the Outer Loop]].
- **The frozen evaluator** as the boundary the agent cannot move — the same back-pressure rule that [[Software Factories, Light and Dark]] makes central to coding factories.
- **Lessons→skills→evals** as the compounding mechanism: review findings graduate into standing guidance with scoped applicability and test cases.
- **Adversarial review with information hygiene** — independence is a property of access, not of session freshness.

## Opinionated take

This is the strongest generalization of the agentic-coding playbook to a different domain I've seen, and it works because data science has something coding lacks: an *independent evaluator* (the holdout, the placebo test) that is cheap, unfakeable, and methodologically mandated. The autoresearcher's 41-experiment run is a clean demonstration that [[Loop Engineering]] patterns transfer when the feedback signal is legitimate.

The weaknesses are the usual playbook weaknesses. The honest numbers are quietly damning — recall of 0.21 on the subgroup that matters means the corrected pipeline is barely usable, and the article doesn't dwell on what you do when the answer to "can we do this?" is no. The adversarial reviewer's blindness is asserted but operationally fragile: in practice, workspaces leak. And the skills-and-evals flywheel assumes someone will do the unglamorous maintenance work of scoping lessons and pruning exceptions — the article hand-waves exactly the part of the loop where most flywheels die.

## Related pages

- [[Causal Inference Agentic Workflow]] — this article cites Netflix's actor/critic division directly; the Netflix note is the production system behind the pattern this article teaches, and both share the fresh-context reviewer discipline.
- [[Own the Outer Loop]] — Osmani's outer-loop/inner-loop split strengthens this source's core claim: specification and verification are the relocated human work, and this article shows the data science version of the same relocation.
- [[Software Factories, Light and Dark]] — the frozen evaluator as back pressure; this article's experiment contract is the same rule ("protect the evaluator from those changes") in analytical form.
- [[Loop Engineering]] — the bounded autoresearcher loop is a domain instance of the loop-engineering taxonomy, with the interesting difference that the evaluator is statistical rather than a test suite.

---
*Sources: [[raw/the-agentic-data-science-playbook]], [[summary/the-agentic-data-science-playbook]]*
*Last updated: 2026-10-03*
