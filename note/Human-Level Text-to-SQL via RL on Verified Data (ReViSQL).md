# Human-Level Text-to-SQL via RL on Verified Data (ReViSQL)

Thinking Machines fine-tunes Kimi-K2.6 with RLVR on expert-verified data and a reshaped reward, hitting human-level text-to-SQL accuracy (92.97% SC-16) without scaffolding, at 12–15% of frontier model cost. The argument is as important as the result: the task knowledge practitioners encode in agentic scaffolds belongs in the training signal, and the binding constraint on RLVR is almost always label noise.

---

Thinking Machines (Zhu, Jin, Choi, Kang) make a claims-backed case against scaffolding as the default answer to domain underperformance. Scaffolded text-to-SQL pipelines (schema-link → generate → self-correct → vote) still lag humans by 11 points on BIRD. Their alternative: fine-tune ReViSQL-K2.6 with RLVR on BIRD-Platinum, a cleaned training set, plus two reward fixes — VeriEQL semantic-equivalence verification (downweighting execution matches that aren't semantically equivalent, which was 32.8% of positive rewards in pilots) and a rule-based process reward forcing the model to translate provided external knowledge into explicit query constraints (24.2% of failures were the model ignoring supplied knowledge). Result: 91.37% greedy at $0.035/task, 92.97% with 16-sample self-consistency at $0.56 — the first text-to-SQL system to exceed the 92.96% human benchmark, beating frontier models at a fraction of the cost, with code and data released.

## Key Quotes

> "Mislabeled instances reverse the signal, degrading learning significantly. Our research found that algorithmic tweaks cannot compensate for this loss — cleaning up the data is crucial for RLVR to work well."

The bluntest claim in the piece, and the one most worth internalising. The audit found 61.1% of BIRD Train instances had at least one error, including 52.1% of the golden queries themselves. Half the "ground truth" was wrong. No RL algorithm rescues a reward signal that is mostly noise — the fix is boring, expensive human verification, not cleverer training maths.

> "The LLM auditor was precise in catching annotation mistakes (90.6% precision) but only caught 24.5% of the errors flagged by humans."

A crisp quantification of the human-vs-auditor gap: LLM reviewers are trustworthy when they flag, but nearly blind to three-quarters of what experts catch. Any "AI cleans the data, human spot-checks" pipeline built on the inverse assumption will quietly ship a poisoned training set.

> "Execution match is not semantic equivalence."

The subtlest insight. A query with a dropped predicate can return the same rows on one database instance and wrong rows on another — so result-based rewards reinforce queries that only look right. Nearly one in three positive rewards was a wrong query. The fix (VeriEQL, <0.1% of training compute) is verification-as-training-signal: the same "oracle over the task, not one instance of it" reasoning that shows up in [[The Oracle Is the Asset]].

> "The task knowledge contained in the scaffold belongs in the training signal, the same training that would uplift the model's overall capacity."

The generalisable thesis. Scaffolding moves human task understanding into prompts the model sees at inference; RLVR on curated data moves it into weights. Same expertise, different home — and the weights version is cheaper, simpler, and doesn't cap out at the base model's ceiling.

## Key Themes

#ai-research #rlvr #reward-shaping #data-quality #text-to-sql #benchmark

## Analysis

The strongest thing here is the honesty of the diagnosis chain. Rather than starting from "RL works," they started from *why* RLVR underperformed humans: label noise first (61% dirty data), then a reward function blind to generalisation and to grounding. Each fix is measured — the 32.8% and 24.2% pilot numbers turn "we thought the reward was bad" into arithmetic. That's rare in this genre.

The result is genuinely a milestone, but read it carefully: 92.97% comes with SC-16 self-consistency, which is sampling 16 candidates and majority-voting by execution result — the authors' footnote insists this isn't a scaffold, and technically it isn't (no separate prompted stages), but it *is* inference-time compute buying the last ~1.6 points. The $0.56 vs frontier pricing comparison is fair; the "no scaffolding" framing is slightly marketing-tinged. The greedy 91.37% at $0.035 is the number that really matters.

The bigger caveat is external. [[Text-to-SQL in the Real World]] argues these benchmarks — even a cleaned one — miss the four structural realities of enterprise warehouses (schema rot, institutional knowledge, contamination). Beating humans on a *cleaned public benchmark* is not the same as answering business questions on a real warehouse; the arc from BIRD to Beaver-like settings remains unproven. What this piece proves is about training methodology, not about production text-to-SQL.

The generalisation claim deserves the benefit of the doubt, though: the same verified-data-plus-shaped-reward recipe transfers to Qwen3-235B and to unseen Spider2 variants and dialects (+12–16% over training on raw BIRD Train). Combined with their earlier financial-judgment work, a consistent Thinking Machines position is emerging: domain expertise is a training-data curation problem, and the frontier model is rarely the right answer for a narrow, verifiable, high-volume task.

## Related Pages

This source directly complicates [[Text-to-SQL in the Real World]]: Stonebraker says the benchmarks measure the wrong thing; Thinking Machines responds by fixing the benchmark's *labels* (Arcwise-Plat-SQL) — a necessary but insufficient reply, since neither piece claims the cleaned public benchmark captures enterprise reality.

It strengthens [[What Broke and Why — RL Post-Training]]'s practitioner autopsy tradition with an enterprise-lab data point on the same lesson: RLVR failures are usually data and reward failures, and "no algorithmic tweak compensates" for bad labels.

It nuances [[The Oracle Is the Asset]] — Ruby's oracle is a test suite humans own; here the oracle is a solver-checked equivalence relation over the *task distribution* rather than one instance, showing what an oracle looks like when the verification target is semantic, not behavioural.

And it sharpens the model-centric branch of [[Three Kinds of Agentic Search]]: the harness-vs-finetune question Turnbull poses has a new datapoint, with fine-tuning winning on cost by a factor of ~10–20× where the task is verifiable and data can be curated.

---
*Sources: [[raw/putting-task-expertise-into-rl]], [[summary/putting-task-expertise-into-rl]]*
*Last updated: 2026-11-17*
