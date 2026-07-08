---
url: https://gwern.net/lean-scaling
title: Lean Software Scaling Laws
author: Gwern Branwen
date_fetched: 2026-07-08
date_published: 2025-09-01
date_modified: 2026-06-23
status: finished
confidence: possible
importance: 7
---

# Lean Software Scaling Laws

**Author:** Gwern Branwen
**Published:** 2025-09-01 (created) | **Modified:** 2026-06-23
**Status:** finished | **Confidence:** possible | **Importance:** 7

## Abstract

The research proposal suggests empirically measuring how coding LLM perplexity scales with codebase size to estimate "scaling laws of 'predictability'" across programming languages. This would translate into security and safety improvements. The idea can be tested either expensively (training from scratch) or cheaply (measuring perplexity over increasingly large context windows). Languages with better scaling exponents "will eventually become easier for LLMs to understand, fix, and write." Lean is singled out as likely having a worse baseline constant now but better scaling exponents, meaning Lean implementations could eventually dominate and "help justify large-scale investments in rewriting existing codebases in Lean."

## Introduction

Coding LLMs are producing most software despite mediocre quality and security issues, with "vibecoded software being especially bad." Future rewrites may help but won't necessarily plug enough holes against LLM-based cyber offensives. The question is whether LLMs could write software in provably secure, formally verifiable ways—and how far away that capability is.

## Language Priors

Neural scaling law methodology remains underused for validating existing approaches. The article notes Luo et al 2025's finding that "programming is especially data-hungry," which could imply long-term lock-in where upgrading to better technologies like Haskell, Rust, or Lean becomes impossible. However, the author argues this doesn't follow: popularity in training data only means LLMs "start off by default performing well."

The piece invokes an "exchange rate" between languages where Python skills partially transfer to others (citing Yang et al 2025), suggesting that "a 'rising tide will lift all boats'" and better coding LLMs could spark a renaissance for obscure languages once models get good enough.

## Scale Failure

Many approaches work at small scales but fail as systems grow. The "BDSM" discipline of languages like Haskell may be painful for quick scripts but indispensable at a million lines of code. LLMs can easily handle short Python scripts that fit in context windows, but what happens as codebases grow? The article asks whether LLMs can keep "dynamic types and exceptions straight" in large, complex, older codebases with monkey-patching and dependency issues. "Because if it can't, even a single error can be fatal."

Lean is described as perhaps the most extreme programming language plausible for large software, having recently seen a burst of popularity for implementing real programs beyond theorem-proving—citing a zlib rewrite in Lean.

## Lean Bottleneck

Rewriting software in Lean could eliminate memory-safety issues, exceptions, out-of-bounds errors, and prove correctness of major properties. However, "there's not much Lean source code out there," so LLMs aren't good enough yet for autonomous translation of large codebases beyond zlib. This creates a chicken-and-egg problem: "We could use LLMs to create large Lean codebases, to train on, so we can replace all our existing code with Lean," but only if those large Lean codebases already existed. It's also unclear whether Lean is well-designed or whether real-world Lean codebases might become a "big ball of mud" with what the author dubs "mathslop."

## Predictability Proxy

The article proposes testing language design quality from an LLM's perspective. A well-designed language should produce codebases that are "easy to read" with few "gotchas"—you could understand a file without tracing dependencies across dozens of others. A well-architected, strongly typed, memory-safe codebase should be "predictable." Conversely, bad design at scale should be increasingly unpredictable.

The author acknowledges that simple compressibility metrics have failed before: "gzip compresses JavaScript the best" and results are unconvincing. What's needed isn't compressibility itself, but perhaps "the second or third derivative of compressibility."

The key insight: good design at scale is not necessarily predictable (since "good design is invisible") but is **increasingly** predictable as you see more of the system. Bad code is increasingly unpredictable—seeing more doesn't help, because "there will be bugs or unintended interactions in even the most immaculate hand-written codebase." The signature of bad proofs (bruteforce cases, reductions) is that "you don't 'learn anything'" because each case is equally predictable.

So LLM perplexity/BPC serves as a weak proxy for system predictability, and languages that yield increasingly predictable large-scale systems will work better with LLMs.

## Measurement Design

The empirical approach uses frozen pretrained LLMs:

1. **Corpus construction**: Take existing source code corpuses by language, create single text files with metadata, possibly dependency-sorted.

2. **Loss measurement**: Run forward passes to measure per-token perplexity across the full context window, normalizing into bytes and per language (since languages can differ 10× in line length). Cross-checks include:
   - **Anomaly/bug detection**: Inject subtle bugs; the more surprised the LLM, the better. Bugs like wrong unit conversions or plausible-but-wrong lemmas require deep semantic understanding.
   - **Correctness**: With artificial context limits hiding more of the codebase, measure how many rollouts still compile and pass tests. For Lean in particular, can an LLM predict a working module from just the module signature header? Do "type signatures constrain the problem space so much that 'there is only one way to do it'?"
   - **Ablations**: Remove/add optional components like type signatures to quantify their benefit.
   - **Semantics**: Identify which parts are especially unpredictable (potentially reflecting bugs) versus predictable boilerplate (potentially reflecting language design defects).
   - **Coreset/minimal context**: A well-designed modular system requires reading only key bits to understand any part. The article compares this to the SAE truesight proposal. Well-designed Lean should require much less context than Python, where "the LLM [might need] to read 10 different files."
   - **Scoring improvements**: Refactors and cleanup that increase predictability provide useful training signal.

3. **Position averaging**: Average by token position.
4. **Curve-fitting**: Fit scaling laws per language.
5. **Extrapolation**: Find crossovers (at what size does Lean become more predictable than Python?) and examine constants/exponents as design metrics.

The author believes this is tractable for a grad student or MATS project.

## Crossover Forecast

The prediction: "weak" languages (dynamic typing, mutable global state, few invariants) will be easier at small scales (thousands of lines) but have worse scaling exponents. They'll "cross over" with strong languages like Haskell at hundreds of thousands or millions of lines. Lean would have the best scaling exponent but a high constant, so it would cross over at some point.

This would motivate large investments in Lean R&D and porting to solve the chicken-and-egg problem by simply paying for training data. One could estimate total training corpuses and the Lean-to-Python ratio to back out how much data would bring the crossover point down to any given repository scale. Exchange rates could suggest whether generating more code in other languages like Haskell would be more cost-effective. "Code can be bought on the market using data-labelers, or by compute using agentic LLMs" to auto-transpile codebases.

## Bias Controls

Initial estimates would be confounded by programmer populations and domains: "Lean code emphasizes math and Lean programmers are highly unusual," while JavaScript involves web dev by ordinary programmers. If JavaScript scaled better than Lean, the approach would need revision. This can be partially controlled by matching codebases by topic and analyzing paired differences. As coding LLMs improve, controlled comparisons could synthesize specifications in two languages and then measure perplexity differences (e.g., original C zlib vs. Lean zlib).

## Context Learning

Languages might systematically differ in in-context learning if LLMs are too ignorant of Lean to use context windows optimally. The author suspects there's enough Lean and that frontier labs emphasize math/programming enough that basic Lean skills are solid, with remaining errors due to smaller training corpuses. This could be checked by finetuning or training from scratch, looking at how the Lean scaling exponent itself scales with training data, though that would be much more expensive.

## Failure Modes

The most likely failure is that "ecosystem maturity and corpus prior effects dominate language-level invariants." That would itself be useful, implying that "tooling, conventions, and documentation beat formal language properties for agentic programming—at least for now."
