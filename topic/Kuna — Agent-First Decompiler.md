# Kuna — Agent-First Decompiler

An experimental decompiler that rivals IDA Pro in control flow structuring — and an LLM wrote nearly every line of the code. Zion Basque's Kuna project is not a decompiler story so much as a software development story: what happens when you give a coding agent benchmark-driven feedback loops and let it study the competition.

---

## Key Quotes

> an LLM has written nearly every line of code in this project

The lede. Basque buries it in the second paragraph, but this is the entire story: not "LLM-assisted," not "AI-augmented" — the LLM *wrote the code*. Every line.

> Kuna achieves perfect structuring on 44.4% of functions, compared with IDA's 45.7%

One percentage point behind the industry gold standard. Not bad for a project whose primary developer can't debug a segfault.

> this is more than just *slop*; it is a truly experimental approach to developing a scientifically interesting tool that gets better automatically

Basque is preempting the obvious dismissal. The defensiveness is warranted — "LLM wrote it" is still a credibility hit in academic circles — but the benchmark numbers do the real work of the rebuttal.

> Kuna needs angr (and other open-source research), and my hope is that, after more time, angr will need Kuna

The ecosystem argument in one sentence. Open-source decompiler research built the training signal; the agent-first tool might eventually feed back into that research. This is not a replacement thesis — it's a symbiosis thesis.

> this is also not *automatic research*; it requires research led by humans

The honest admission that "autonomous refinement" still requires humans to discover the right metrics, curate the benchmark data, and define what "better" means. The LLM optimizes within a human-designed fitness function. This is the key constraint on the whole experiment.

## Key Themes

#decompiler #agentic-development #benchmark-driven #open-source #reverse-engineering #compiler-design #rust #autonomous-refinement

Kuna is a Rust port of Ghidra, reworked to match angr's pipeline. An LLM (likely Claude, given UGA/AFRL/Metalware connections) reimplemented more than 20 fundamental features from angr — features that took years of PhD-level research to design — through trial-and-error refinement against benchmarks. The loop: run benchmarks, find where Kuna trails IDA Pro or Ghidra, let the LLM study the difference, repeat.

The project sits at the intersection of three threads:
- **Decompiler research.** Structuring, type recovery, variable identification — hard compiler-science problems that have occupied researchers for decades. Kuna addresses structuring; the rest remains.
- **Benchmark-driven agentic development.** Not vibe coding. Not "build me a decompiler." A tight feedback loop where every refinement is measured against decbench.com results.
- **Open-source ecosystem symbiosis.** Kuna wouldn't exist without angr and Ghidra. The question is whether it can become useful enough to repay the debt.

## Critical Analysis

**The benchmark is the whole game.** Without decbench.com — fundamental metrics that, as Basque notes, "have only emerged in the last few years" — this experiment doesn't work. The LLM had a crisp, numeric target to optimize against. Most domains don't have that. This is both the experiment's strength and its limited generalizability: Kuna works because decompiler output quality is measurable in a way that, say, "is this code maintainable?" is not. [[Harness Engineering is not Enough]] makes the same point from the other direction: RL can't reward maintainability because its cost function plays out over months.

**The "LLM wrote every line" framing is doing a lot of work.** Basque is careful to credit Ghidra (the codebase Kuna ports), angr (the pipeline it mimics), and the research community (the metrics it optimizes against). The LLM didn't design a decompiler — it ported one architecture to match another's pipeline, guided by benchmarks. That's genuinely impressive but it's a different claim than "AI invented a decompiler." The LLM is doing the implementation work; humans set the destination. This is [[Specifications as the Product]] in the most literal sense: the architecture spec and benchmark suite are the durable artifacts; the code is disposable.

**The structuring gap is nothing; the rest is everything.** 44.4% vs. 45.7% is a rounding error. If Kuna never improves structuring, it's already a viable tool on that dimension. But types, variable identification, and recompilability are where decompilers earn their keep in real reverse engineering. Basque is honest that these remain open questions. The experiment's real test is whether the autonomous-refinement loop works on problems where the right answer isn't as crisply measurable as "does the control flow graph match?"

Those unsolved dimensions are exactly what [[bt-re-controller — Bluetooth Firmware RE Skill]] attacks from the *other* direction — leaving the decompiler alone and layering an LLM + spec-reference-table pipeline on top to recover names, arg types, and per-connection structs. If Kuna is "teach the machine to build the tool," bt-re-controller is "teach the machine to be a senior RE on top of the tool."

**The open question is the most interesting one.** Can this framework — LLM + benchmarks + refinement loop — produce a decompiler that's genuinely competitive across all dimensions? If yes, it's a proof point for [[Loop Engineering]] as the meta-skill: the human designs the improvement loop, not the code. If no, it's evidence that some problems resist optimization-by-benchmark, and you still need human insight to crack them.

**The symbiosis thesis is underappreciated.** Basque's "angr will need Kuna" line isn't just nice — it's the only way this experiment closes the loop. If Kuna only consumes open-source research without contributing back, it's parasitic. If the agent-first approach produces insights that feed back into angr's research program, it's mutualistic. Which one it becomes depends on whether "autonomous refinement" surfaces patterns that are legible to human researchers, or just produces a tangle of weights that happen to pass benchmarks.

**The "not slop" defensiveness is a sign of the times.** In mid-2026, anyone releasing an LLM-built tool has to preempt the slop accusation. Basque does it with benchmark numbers. That's the right move — but it also reveals that the burden of proof has shifted. Human-built tools are presumed competent until proven otherwise; LLM-built tools are presumed slop until they post numbers. Whether that stigma fades or hardens will determine whether Kuna is a one-off curiosity or the first of many.

---

*Sources: [[raw/kuna-release]]*
*Last updated: 2026-08-01*
