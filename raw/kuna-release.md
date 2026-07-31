---
url: https://noelo.org/blog/kuna-release/
title: "Kuna: Decompiler Development in the Age of Coding Agents"
author: Zion Leonahenahe Basque
date_fetched: 2026-08-01
date_published: 2026-07-29
---

# Kuna: Decompiler Development in the Age of Coding Agents

**Author:** Zion Leonahenahe Basque, Assistant Professor, School of Computing, University of Georgia
**Published:** July 29, 2026 on the Noelo blog
**Context:** Work conducted as a visiting faculty researcher at the Air Force Research Lab (AFRL) and a research fellow at Metalware.

## Summary

Zion Leonahenahe Basque announces the release of Kuna, an experimental "agent-first decompiler designed for autonomous refinement." The central surprise: an LLM wrote nearly every line of code in the project, yet Kuna rivals IDA Pro 9.2 in control flow structuring on C programs.

Per recent benchmark results at decbench.com, Kuna achieves perfect structuring on 44.4% of functions, compared with IDA's 45.7%. This performance was largely achieved through autonomous refinement: the LLM studies examples where it performs worse than IDA Pro on fundamental metrics, allowing it to effectively learn how another decompiler solves hard problems through trial and error.

The experiment also involved Ghidra and the angr decompiler — Basque spent his entire PhD as a core developer of the latter. Through this method, an LLM reimplemented in Kuna more than 20 fundamental features from angr, features that took years to design via scientific advancements in decompilation. Basque stresses the project is "more than just slop" — it is a truly experimental approach to developing a scientifically interesting tool that gets better automatically.

The post includes a side-by-side image comparing Kuna's and Hex-Rays' decompiled output for a popular angr switch feature.

## Three Limitations

1. **Debt to prior research.** Kuna is only possible through decades-long efforts of decompiler researchers. Technically, Kuna is a Rust port of Ghidra, reworked to more closely match angr's pipeline. Basque has no plans to leave angr, which remains the first place to go for developing frontier algorithms. Kuna is an experiment in what high-level scientific feedback alone can achieve. "Kuna needs angr (and other open-source research), and my hope is that, after more time, angr will need Kuna."

2. **Human scientific insight is indispensable.** "Automatic" refinement required discovering new fundamental metrics, studying what aligns with human reversing values, and years of insight into what data a meaningful benchmark requires. "This is also not automatic research; it requires research led by humans."

3. **Much work remains.** "We are doing well on structuring, but decompilation is more than just structuring!" Remaining gaps include types, optimizations, recompilability, and variable identification. The open question is whether all of those can be achieved within Kuna's development framework.

## Acknowledgments

Basque credits PhD advisors Fish (Ruoyu Wang) and Yan, plus insights from Metalware, AFRL, and the Department of Defense. Code is on GitHub; a more technical follow-up post is promised.
