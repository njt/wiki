# Simmer Skill

A Claude Code skill from 2389 Research that implements iterative artifact refinement via an agentic judge-generator loop. Give it anything text-shaped and a set of criteria, and it generates, scores, feeds back prioritized improvements, and repeats — converging in 3-5 rounds instead of thousands. Its authors tested it by having it improve *its own skill definition* through an inner/outer agent architecture.

---

## Key Quotes

> "Pointed feedback plus a capable agent means you converge in three to five rounds instead of three thousand."

This is the core thesis and it's a good one. The model already knows what good looks like — it just doesn't know what's missing from *this specific artifact*. You don't need RL-scale iteration; you need targeted critique. ASI (Actionable Side Information) is the name for feedback that's specific enough to act on.

> "Judges need calibration or they inflate: early runs saw all scores at 9.2 despite poor output quality."

The most practically useful finding in the piece. Without calibration anchors, LLM judges drift toward politeness. The fix — giving the judge the seed artifact and its iteration-0 scores as permanent context — is a lightweight calibration pattern applicable to any eval setup. This is the same problem [[Benchmark Exploitation]] documents at the benchmark level, showing up inside a single refinement loop.

> "Replace the instruction with an explicit contract."

The skill definition improved faster than the artifacts it produced. Why? Because ambiguous instructions caused agents to diverge on iteration counts, table schemas, and process details. Replacing prose instructions with explicit contracts eliminated the variance. This is the same insight as [[Feedback Loop is All You Need]] applied to skill design: deterministic structure beats natural-language pleading.

---

## Key Themes

#tool #agentic-coding #quality #iteration #claude-code-skill

### The Self-Honing Meta-Loop

The most interesting thing about Simmer isn't what it does — iterative refinement is a well-understood pattern (see [[Trycycle]], [[Designing Agentic Loops]]). It's that the authors used Simmer to improve Simmer. The outer loop evaluated the skill definition against test tasks; the inner loop ran three independent agents per version to measure consistency.

This is [[Self-Distillation]] applied to process rather than output. The skill doesn't just refine artifacts — it refines the instructions that produce artifacts. Each meta-iteration makes the skill more specific, more consistent, and less dependent on agent interpretation.

### Judge Calibration as the Hard Problem

Everyone who builds agentic evaluation loops eventually discovers that LLM judges are too nice. Simmer's calibration fix is simple but not obvious: give the judge the *original* artifact and its *original* scores as permanent anchoring context. This prevents score drift without requiring explicit rubrics (which LLMs also drift from, just more slowly).

This calibration pattern should be standard in every eval framework. It costs almost nothing in tokens and prevents the "everything is 9.2" failure mode.

### Explicit Contracts Over Prose Instructions

The finding that the skill improved faster than the artifacts is genuinely surprising and important. It means the bottleneck in agent quality isn't the agent's capability — it's the clarity of the specification. This echoes [[How to Write a Good Spec for Agents]] and [[Specifications as the Product]]: the durable value is in the spec, not the output. Simmer's self-improvement loop is really a spec-improvement loop.

### Where It Fits

Simmer occupies a specific niche in the refinement-tool landscape:

- **[[Trycycle]]**: Plan-strengthen-review with fresh agents at every stage. Broader scope (full development workflow), more expensive (13+ invocations), but benefits from disposability.
- **Simmer**: Narrower scope (artifact refinement only), tighter loop (3-5 rounds), self-improving via meta-iteration. Lighter weight for text artifacts specifically.
- **[[Compound Engineering]]**: The philosophical parent — add a system rather than doing manual review. Simmer is compound engineering packaged as a skill.

---

## Critical Analysis

**What's real:** The judge calibration insight is immediately useful and under-discussed. Every team running LLM-as-judge evals should steal the anchoring-context pattern. The 3-5 round convergence claim is plausible given pretrained competence — the model already knows what good looks like, it just needs pointing.

**What's undersold:** The self-honing meta-loop is a bigger deal than the post makes it sound. "We used the skill to improve the skill" sounds cute but it's actually a generalizable pattern for any skill or prompt that produces evaluable output. You don't need an outer agent loop to do this — you can just run the skill against test cases, judge the output, and feed the critique back into the skill definition. This is skill-level [[Self-Distillation]] and it should be standard practice.

**What's missing:** The post doesn't address when Simmer fails. What kinds of artifacts resist iterative refinement? When does the judge's calibration drift even with anchoring? What's the failure mode when the criteria themselves are wrong? These matter because the skill's pitch — "works on anything text-shaped" — implies a generality that probably has sharp edges.

**The marketplace context:** Simmer is part of [[2389 Plugin Marketplace]], which positions it as a packaged capability rather than a technique. The packaging matters because it turns "run an agentic refinement loop" from something you have to design into something you can invoke. That's the marketplace thesis in microcosm: skills as shrink-wrapped [[Designing Agentic Loops]].

---

*Sources: [[summary/simmer-skill]], https://2389.ai/posts/simmer-skill/*
*Last updated: 2026-05-15*
