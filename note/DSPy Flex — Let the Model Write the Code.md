# DSPy Flex — Let the Model Write the Code

DSPy Flex is a module that lets optimizers rewrite not just a program's prompts but its *code*, producing programs that are simultaneously more accurate, cheaper, and faster than hand-written or prompt-only-optimized alternatives. It operationalizes the thesis that models are now good enough at programming to continually compile our harnesses — balancing what's deterministic code and what's an LLM call — as new data, models, and tactics arrive.

---

## Key Quotes

> "Flex is just a Predict module (or RLM if you provide tools). What makes Flex different is what it exposes to an optimizer: Flex exposes its code, in addition to its instructions."

The architecture is intentionally simple. A `dspy.Flex(Signature)` is a drop-in replacement for `dspy.Predict(Signature)` — same interface, same behavior before optimization. The difference is what the optimizer can see and modify. This design choice is worth noting: the module doesn't *do* anything special at runtime; it *exposes* more surface area to the optimizer. The power is in the interface, not the implementation.

> "Sometimes it doesn't call the model at all, because it found a case it could settle in code. And when it does call, the call is better aimed, because the module has already done the parsing and comparison and hands the model a narrower question."

The dual benefit. Fewer calls and better calls — not one or the other. The generated code pre-processes, normalizes, and bins inputs so that the model only sees the genuinely ambiguous cases, and sees them with computed features (similarity scores, parsed addresses, distance) rather than raw text. This is the architectural inversion [[Reducing Token Spend with Deterministic Workflows]] advocates, but discovered by the optimizer rather than designed by the engineer.

> "Even with calls free (λ=0), the optimizer wrote code. The metric function scored on accuracy only. The only nudge was in the metric's textual feedback asking for cases to be settled in code where possible."

The most surprising result. With no cost penalty, optimizing for accuracy *alone*, the reflection model still chose to route 75% of records through deterministic Python — and came out *more accurate* than calling the model every time (95.0% vs. 90.4%, p=0.019). The rules handle easy cases better than a small model, and the model only sees cases that actually need judgment. This is an empirical argument against the "just call the model for everything" instinct: for classification tasks, deterministic pre-processing is not just cheaper but *more accurate*.

> "The comment at the top, 'the LLM is a LAST-RESORT fallback,' was written by the optimizer about its own architecture."

The optimizer describing its own design decision. This meta-commentary is striking because it shows the reflection model isn't just generating code — it's making and naming architectural choices. The three-stage normalize-compare-decide pipeline with a model-only-as-judge endpoint is a pattern a human engineer would recognize. The model chose it independently.

> "As we've watched GEPA rewrite programs across many types of tasks, four moves keep showing up: Decomposition, Method selection, Routing, Evolution."

The recurring pattern language. These aren't optimizations — they're architectural refactorings. The optimizer is doing the kind of work a senior engineer does when they look at a monolithic `Predict` call and think "this should be a pipeline." The difference is it does it automatically, guided by a metric.

Shopify's [[Sidekick's Continual Learning Loop]] names GEPA (alongside Agentic Context Engineering) as its judge-calibration optimizer in production — the same reflective prompt evolution, applied to a live merchant agent rather than a benchmark. Real-world confirmation that GEPA's moves generalize past DSPy's eval sets.

---

## Key Themes

#pattern #tool #concept

- **#pattern — Code as the second lever**: Prompt optimization has one degree of freedom (the instruction text). Adding code as a second lever changes the shape of what's optimizable — from "better words" to "better architecture." The optimizer can decompose, route, write helper functions, and choose between code and model per step. This is the step beyond [[Prompt Debt]]'s "search, don't write" prescription: search the space of programs, not just the space of prompts.

- **#concept — The deterministic-to-model spectrum as an optimization target**: Every AI system makes implicit choices about what goes to code and what goes to the model. Flex makes those choices explicit and optimizable. The λ penalty knob lets you dial from "accuracy at any cost" to "never call the model," with the optimizer finding the best program at each point. This is the same tradeoff [[Reducing Token Spend with Deterministic Workflows]] enforces through human design; Flex enforces it through metric design.

- **#pattern — Three-stage normalize-compare-decide**: The generated code consistently follows this structure. Normalize strips noise (legal suffixes, franchise numbers, generic words). Compare bins into confident-match / confident-miss / unsure. Decide applies rules per bucket, with the model as fallback. This isn't a trick — it's a generalizable decomposition pattern the optimizer discovered across tasks.

- **#concept — The model as last-resort judge**: In the generated program, the LLM is not the primary decision-maker. It's the tiebreaker for cases where deterministic signals genuinely conflict. This inverts the default architecture (model-first, rules as guardrails) to rules-first, model as exception handler. [[Guardrails and Feedback Loops]] argues for deterministic enforcement over prompt-based constraints; Flex operationalizes this at the architecture level.

- **#concept — Continual harness compilation**: "We can continually compile our harness, balancing what's written as code and what's handed off to an LM, as new data, models, and tactics arrive." This positions the harness not as a static artifact but as something that should be recompiled — like code, not like configuration. When a cheaper model ships or your dataset grows, recompile. The harness becomes a function of (data, model, metric) rather than a fixed design. This is [[Harness Engineering for Self-Improvement]] made operational.

---

## Critical Analysis

**The benchmark is honest and well-designed.** Caches disabled throughout (cold production numbers), class-balanced eval set (50% is chance), McNemar test for statistical significance, mean latency under realistic 8-way concurrency. This is a model for how to report optimization results. The raw numbers are small (240 records in the eval set) but the methodology is sound.

**The "even with calls free" result is the most important finding and needs replication.** If it holds across tasks, it means the instinct to route everything through the model is not just expensive but *less accurate* for a meaningful class of problems. The mechanism makes sense — deterministic code doesn't hallucinate on easy cases — but one benchmark on one task with one model pair (Haiku/Opus) is a proof of concept, not a law. The claim is strong enough to warrant skepticism.

**The λ penalty is doing two things at once, and the paper is honest about it.** The explicit mechanism is the score penalty: `score = max(0.0, correct - λ × n_llm_calls)`. The implicit mechanism is the metric's *textual* feedback nudging toward code. The authors note this explicitly, which is good methodology — but it means you can't cleanly separate "the optimizer learned to use code" from "the optimizer was told to use code." The textual feedback is a prior that shapes the search space.

**The sandboxing claim is necessary but under-specified.** "It never runs in your process" is the right instinct — generated code is untrusted — but the article doesn't detail the sandbox architecture. What's the interpreter? What's the attack surface of the bridge (predictor calls and provided tools)? For a feature that modifies its own execution environment at optimization time, the security model deserves more than two sentences. [[How We Contain Claude]] and [[Bounding the Blast Radius]] provide the framework for thinking about this; Flex's sandbox should be evaluated against those standards.

**The SWE-bench Pro pilot is intriguing but thin.** 0/12 → 4/12 with 60 metric calls is a real improvement, but 4/12 on a 12-problem sample has a wide confidence interval, and the comparison to "39% Haiku is reported to reach inside a mature, hand-built harness" is an apples-to-oranges comparison (different problem subsets, different evaluation conditions). The pilot is a direction, not a result. That's fine — the authors present it as such — but it shouldn't be cited as evidence until replicated.

**The relationship to [[Prompt Debt]] is direct and important.** Breunig argued that prompts should be searched, not written, and cited DSPy/GEPA as proof. Flex extends this argument from prompts to *programs*. If prompt debt comes from hand-tuning natural language against probabilistic models, code debt might come from hand-writing routing logic and decomposition against changing data. The prescription is the same: specify behavior with measurements and let the optimizer search. Flex makes the search space bigger.

**The relationship to [[Harness Engineering for Self-Improvement]] is deep.** Flex + GEPA is a worked example of Weng's "harness as optimization target" thesis. The four moves (decomposition, method selection, routing, evolution) are exactly the harness design decisions Weng surveys researchers trying to automate. Flex does it today, on real tasks, with a reflection model and a metric. The difference is that Flex optimizes the *task-specific* harness (the module code), not the meta-harness (the optimization framework itself). That's the next step: a Flex module whose code *is* the optimizer.

**What's missing: the cold-start problem.** Flex requires a metric — something that can score the program's output. For classification and extraction tasks with labeled data, that's straightforward. For open-ended generation, summarization, or creative tasks, the metric is the hard part. [[Eval-Driven Development (Airbnb)]] provides the methodology (calibrated LLM-as-judge, programmatic checks, human evaluation), but the cost of building that metric infrastructure is real. Flex democratizes optimization for metric-rich tasks and leaves metric-poor tasks where they are.

**The most exciting implication is the one the article only gestures at: competitive dynamics.** If the best program for a task is a function of (data, model, metric), then every time a new model ships or your dataset grows, you should recompile. The program that was optimal for Haiku 4.5 might not be optimal for Haiku 5.0. The program that was optimal with 1,000 training examples might not be optimal with 10,000. This turns harness design from a one-time engineering investment into a continuous process — and makes the metric the durable asset, not the program. [[The Oracle Is the Asset]] applied to the harness itself.

---

*Sources: [[raw/let-the-model-write-the-code-html]], [[summary/let-the-model-write-the-code-html]]*
*Last updated: 2026-08-08*
