# Harness2 — Recursive Agent Harnessing for an Open World

Google Cloud AI Research's Harness2 (arXiv:2610.14224, Xu et al., 2026-10-09) tackles the hardest version of harness self-improvement: refining a harness at test time on novel tasks with **no ground-truth verifiers**, while making the improvement process itself accumulate across a task stream. The framework couples two recursions — task-level refinement of the current execution harness, and harnessing-level learning that updates the *improvement harness* from the refinement experience.

---

The key problem statement: early harness-RSI methods optimize against fixed training tasks or unit tests, and their statically-optimized harnesses fail to generalize to unpredictable open-world tasks. But moving RSI online creates a dilemma — tasks lack ground truth, and task-specific edits don't transfer when requirements change. Harness2's answer is that "the refinement process itself produces highly reusable experience: how a weakness is diagnosed, which changes are tested, and what their execution reveals." So instead of inheriting the previous task's tailored harness, each new task starts from a common base H0 and is guided by a library of adopted harnesses plus EXPERIENCE.md lessons.

The contrastive **Propose–Probe–Compose** step is the load-bearing idea. It proposes an *invariant* candidate (targeting recurring agentic deficiencies: planning, tool use, context management, delegation) and a *domain-specific* candidate (targeting the task domain's knowledge and constraints). Each is executed under matched conditions against a reference execution, and composition keeps only edits supported by behavioral contrasts — evidence of what changed, not a presumed score. The paper is explicit that these contrasts "inform subsequent composition rather than verify task correctness, as execution stochasticity and shared model biases limit the reliability of the evidence." That epistemic honesty is what separates this from test-time methods that regress (Harness Scaling underperforms base in 7 of 12 configurations).

The harness is decomposed into eight file-level editable components — system prompt, guardrails, memory, skills, tool guidance, scripts, subagents, and a middleware plugin — extending AHE's file-level scheme by one. Each is mounted as a file in the runtime, which is what makes an LLM editor able to diff and edit it.

Results: one improvement step gains up to 11.5 points on WorkBuddy-Bench, 4.5 on JobBench, and 11.8 points on Harvey's LAB across GLM-5 and Gemini 3.5 Flash under OpenCode and Codex (edited by Claude Opus 4.8). Weaker executors benefit more — GLM-5 with an improved harness overtakes base Gemini 3.5 Flash — and the same model playing executor and editor still improves on every benchmark. Libraries built with bigger k budgets strengthen future tasks without further probing, and even transfer across benchmarks (JobBench → WorkBuddy-Bench).

---

## Key Quotes

> "The reference reveals how the harness behaves, not whether its deliverable is correct."

The methodological pivot of the whole paper. Without ground truth you cannot score outputs, but you can still observe *behavior*. Harness2 converts the verifier problem into a differential-diagnosis problem — the same instinct behind A/B testing and ablation in engineering. Contrast this with ReasoningBank, which in their runs fell as much as 6.6 points below base on coding tasks, presumably because unverified reflections can steer the wrong way.

> "Lessons are retained only when supported by execution contrasts; selecting an output alone does not establish transferability."

The library's admission rule. This is the discipline that keeps harnessing-level recursion from becoming an echo chamber: an entry must carry evidence that a change caused an observable behavioral difference, not just that an output was chosen. It's a memory design pattern worth stealing for any agent that writes its own [[Agent Memory]].

> "Selection adds no harness revision. At k = 1, both forms reduce to the basic contrastive step... The corresponding harness is denoted H★, where the star indicates adoption rather than verified optimality."

A quiet but important honesty marker in the notation: the adopted harness is not claimed optimal, merely selected. The paper keeps refusing to blur evidence and preference.

> "A weaker model with an improved harness can thus outperform a stronger one with its original harness."

The harness thesis in its strongest empirical form, and a direct corollary of the harness-definition line in [[Harness Engineering for Self-Improvement]]: the harness is where the missing capability actually lives. It also reframes model choice economics — a cheap model plus a good harness library may beat an expensive model bare.

> The case study: the base run "revealed procedural failures rather than knowledge gaps: data was never reconciled across source files, and outputs were never reopened for verification after being written." The invariant edit forced inventory of all inputs and reopening produced files with a verification script: "Confirming a file exists is not confirming it is correct." Score went 43.1 → 77.8.

The recurring diagnosis is that agents fail *procedurally, not knowledge-wise* — which is exactly why harness edits (guardrails, verification scripts, skills) fix them. The guardrail language here is verbatim the same medicine prescribed by practitioner essays like [[Agentic Verification]] and [[An Honest Review of AI Programming]].

---

## Key Themes

- #concept — **Recursive agent harnessing**: separating the execution harness (what runs) from the improvement harness (what refines it), and letting each recursion level feed the other.
- #pattern — **Contrastive improvement without verifiers**: Propose–Probe–Compose replaces ground-truth scoring with behavioral differential evidence.
- #pattern — **Scoped proposals**: invariant vs. domain-specific editors as a division of labor; ablation shows scoping alone is worth 2.9–3.7 points.
- #research — Evidence that harness gains accumulate: larger exploration budgets produce libraries that improve future tasks even without further probing, and libraries transfer across domains.

## Critical Take

The strongest part of this paper is its epistemology. Most self-improvement work quietly assumes a verifier or a reward signal and Optimizes™; Harness2 instead asks what can be learned *without* one, and the answer — behavioral contrast under matched conditions — is genuinely principled. The admission rule for library entries (no contrast, no lesson) is the right guardrail against the known failure mode of reflection-based memory, and the cross-benchmark library transfer is the kind of result that suggests the abstraction is real rather than benchmark-shaped.

The caveats are real though. The benchmarks are professional workflows (office deliverables, job tasks, legal work) judged partly by rubric-like pass rates; "Partial" pass rate on LAB is a coarse metric and a +11.8 jump invites scrutiny of what the judge rewards. The improvement harness's stage prompts are fixed, so the meta-level is still human-designed — this is recursion at two levels, not open-ended self-modification. And compute: k=5 parallel recursion multiplies execution cost per task; the Pareto-frontier claim is honest but the absolute token counts per task run into the tens of millions, which puts this firmly in the research regime rather than the everyday practitioner's loop. Finally, gains are concentrated where base harnesses are weak — the GLM-5 story — which means at least part of the headline is "bad harness + good editor" recovering ground, not frontier harnesses getting better.

Still, the direction is right and aligns with the thesis in [[Harness Engineering for Self-Improvement]]: near-term RSI lives in the harness layer, and the open problems Weng flags (weak evaluators, safe use of imperfect evidence) are exactly what the contrastive step is designed for.

---

*Sources: [[raw/2610-14224]], [[summary/2610-14224]]*
*Last updated: 2026-10-20*
