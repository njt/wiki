# AI Needs to Think Before Giving Feedback

Luke Shepard argues that one-shot AI feedback is unreliable — too wordy, emotionally mismatched, or factually wrong — and that a structured **G-E-RG loop** (Generate → Evaluate → Re-Generate) transforms sloppy feedback into something teachers and students can actually trust. The insight generalizes beyond education: any domain where ground truth is fuzzy benefits from having the AI evaluate its own output before delivering it.

---

## Key Quotes

> "For teachers to get real leverage from AI agents, the systems need to be more accurate and reliable than one-shot attempts can deliver."

Shepard is pushing back against the "ship the first draft" instinct. In education, the cost of bad feedback isn't a failed build — it's a student who internalizes wrong information or a teacher who stops trusting the tool entirely.

> "Regardless of initial quality, through evaluation and regeneration, it can be transformed into high-quality feedback."

This is the Stanford paper's core finding (Cao & Koedinger, AIED 2024). The Evaluate step is the differentiator — it's not that the initial generation is always bad, it's that the loop makes it *consistently* good. Stability matters more than peak quality when you're deploying to real classrooms.

> "They don't have to be perfect, just good enough to be able to gradually earn trust of teachers and students over time."

The bar isn't perfection — it's better-than-nothing with a trajectory. This mirrors the agent coding world's relationship with unit tests: the feedback mechanism doesn't need to catch everything, it just needs to catch enough to make the loop converge.

---

## Key Themes

- **#pattern** G-E-RG loop — Generate, Evaluate (with a rubric and different LLM), Re-Generate. A domain-agnostic quality improvement pattern that works wherever you can articulate evaluation criteria.
- **#concept** Feedback criteria formalization — Good feedback should be reliable, include critiques *and* strengths, suggest actionable next steps, strengthen relationships, and encourage agency. Most AI feedback fails on the emotional dimension.
- **#concept** The squishy-domain problem — Coding has unit tests; education has... teacher judgment. LLMs can serve as approximators of that judgment, enabling automated feedback loops in domains where deterministic checks are impossible.
- **#pattern** Structured iteration beats raw capability — LLMLoop showed 71% baseline accuracy jumping to over 90% with 10 structured retries. This is the same dynamic as [[Feedback Loop is All You Need]]: the loop matters more than the model.
- **#tool** DeepAnalyze — Chinese research team's model using special tokens to analyze, reflect, and revise iteratively. Consistently outperforms all compared systems on open-ended research tasks.

---

## Critical Analysis

**What's right:** Shepard correctly identifies that the G-E-RG pattern is domain-agnostic. The same structure — generate, evaluate against criteria, regenerate with context — works for code review, writing feedback, lesson planning, or any output where quality is multidimensional and hard to capture in a single pass. The connection to [[Designing Agentic Loops]] is direct: Willison names the meta-skill; Shepard provides a concrete instantiation with empirical backing.

**What's underargued:** The article treats "use a different LLM for evaluation" as a minor implementation detail. It's not — it's the whole ballgame. Same-model evaluation suffers from [[Fresh Eyes]] blindness: the model that made the error is unlikely to catch it. The Stanford paper's Evaluate step works precisely because it introduces an independent perspective. This is the same reason [[Trycycle]] uses fresh agents at every stage.

**The missing dimension:** Shepard mentions coding benchmarks but doesn't connect to the [[Harness Engineering]] framework. In Böckeler's terms, the Evaluate step is *inferential feedback* (an LLM judging quality) while unit tests are *computational feedback* (deterministic pass/fail). Education needs inferential feedback because there's no compiler for "good teaching." But that makes it inherently less reliable — you're stacking one probabilistic system on top of another. The article acknowledges this ("they don't have to be perfect") but doesn't grapple with the compounding error problem.

**The commenters saw further:** Adam Lupu's flipped framework — have students evaluate drafts themselves — inverts the loop in a way that might be more pedagogically valuable than any AI-generated feedback. Learning to evaluate *is* learning. Priya Mathew Badger's observation about coding-writing parallels suggests a research program: what transfers from GitHub's AI code review systems to classroom writing feedback, and what doesn't?

**Bottom line:** The G-E-RG pattern is real and useful, but it's one instance of a deeper principle that's scattered across this wiki: [[Compound Engineering]], [[Guardrails and Feedback Loops]], [[Don't Fear the Dark Factory]]. The unifying idea is that **generation is the easy part; the system that checks the generation is where the engineering lives.**

---

*Sources: [[raw/ai-needs-to-think-before-giving-feedback]]*
*Last updated: 2026-05-15*
