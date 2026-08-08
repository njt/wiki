---
url: https://www.cmpnd.ai/blog/let-the-model-write-the-code.html
date_fetched: 2026-08-08
---

*This is a guest post by Michael Isaac, a PhD student in software engineering at Carnegie Mellon University, written during his internship at cmpnd. Michael implemented Flex, the module this post introduces, for DSPy.*

The core position of DSPy is that you can define a task once, in a way that lets it be re-implemented as the AI ecosystem advances. The history of these re-implementations can be understood as a history of the models; of weaknesses we worked around and strengths we leveraged:

- In 2022, models needed to be shown what a task looked like, so optimizers like BootstrapFewShot automated the picking of few-shot examples.
- Models then grew to be capable prompt authors, so optimizers like MIPROv2 and GEPA could improve programs by rewriting their instructions.
- Lately, models have become excellent programmers.

This week, we're introducing `Flex` to DSPy, which leverages the coding skills of models to rewrite not just the instructions of your program, but the code itself.

## Flex Lets GEPA Optimize the Code

`dspy.Flex(YourSignature)` is a DSPy module, which can be dropped into any of your existing `Predict`, `ReAct`, or `RLM` programs. For example:

```
my_signature = "question -> answer"
my_program = dspy.Predict(my_signature)
# Make it Flex!
my_program = dspy.Flex(my_signature)
```
If we run either of these programs, we'd get the same result. Prior to optimization, Flex is just a Predict module (or RLM if you provide tools).

What makes Flex different is what it exposes to an optimizer: `Flex` exposes its code, in addition to its instructions. Hand a Flex module to `dspy.GEPA` and the reflection model might decompose your program, write helper functions, implement routing logic, *and* rewrite your prompts. The output is an optimized program that performs best against the metric you gave it.

This is how you'd optimize a Flex module with GEPA. `SamePlace` here is the signature for the location conflation task we'll walk through in the next section:

```
program = dspy.Flex(SamePlace)        # was: dspy.Predict(SamePlace)
# cheap LM to use during inference
dspy.configure(lm=dspy.LM("anthropic/claude-haiku-4-5"))
# big LM to write the code and instructions
big_lm = dspy.LM("anthropic/claude-opus-5")
optimized = dspy.GEPA(
    metric=make_metric(penalty=0.2),
    reflection_lm=big_lm,
    max_metric_calls=400,
).compile(program, trainset=train, valset=val)
```
After optimization, `optimized.save("program.json")` persists the source and `dspy.Flex(SamePlace).load(...)` restores it. The artifact is a file you can open, read, diff, and reason about.

What you get back is a program the reflection model wrote to score as high as it can against your metric. Two things tend to follow. Sometimes it doesn't call the model at all, because it found a case it could settle in code. And when it does call, the call is better aimed, because the module has already done the parsing and comparison and hands the model a narrower question. Fewer calls, better calls, and a program that outperforms the one you handed it.

Code written by a model is still untrusted code, so by default **it never runs in your process**. Flex executes the generated source inside a sandboxed interpreter. Only predictor calls and the tools you explicitly provided bridge back to the host process, and a `max_predictor_calls` cap bounds how many times per forward that bridge can be crossed.

## Location Conflation Task

Last year, Drew demonstrated prompt optimization at the Data + AI Summit with a geospatial conflation task: given two place listings, decide whether they're the same physical place. It's deceptively hard in the tail. KIN CAFE and KIN at the same address are the same place. CONCESSION #2 KEN MERCER SPORTS PARK and KEN MERCER SPORTS PARK at the same address are not.

We replaced `Predict` with `Flex` and ran GEPA on this task: 1,029 labeled pairs, evaluated on 240 held-out records (class-balanced, so 50% is chance). Caches were disabled throughout, so the cost and latency figures below are what cold production traffic would pay.

The **baseline** is the original `dspy.Predict`, one model call per record: 90.4% accuracy at $0.98 per thousand records.

**Optimizing only the prompt** with GEPA, no Flex, lifts accuracy to 92.5%. But the only lever a prompt optimizer has is the instruction, so it wrote a much longer one, and every record pays for those extra tokens at inference: $2.88 per thousand records, 2.9x the baseline cost, and 48% slower.

**Flex gives the optimizer a second lever: the module code.** Running GEPA, unchanged, on the Flex program lifted accuracy from 90.4% to 95.0% at a cost of $0.70 per thousand records. By optimizing the prompt *and* the code, Flex produced a program that is 28% cheaper and 40% faster than the baseline.

How is this possible? For one, many of the places being compared can be evaluated using only code. Our reflection model wrote code to identify and route these easy matches to plain Python functions, resulting in *75% fewer LLM calls*.

We can lean into this behavior by updating our metric. A GEPA metric returns a score plus natural-language feedback. With Flex, it can also see how many LM calls the generated program made on each record. We can use this value to penalize our feedback:

`score = max(0.0, correct - PENALTY * n_llm_calls)`At PENALTY = 0, calls are free, and the optimizer chases accuracy alone. As the penalty (λ from here on) rises, every LM call has to buy back more accuracy than it costs, and the optimizer is pushed to settle cases in Python and reserve the model for genuine ambiguity. (Past λ = 1.0, a call can never pay for itself; that end of the dial means never calling the model.) We swept λ across 0, 0.05, 0.1, 0.2, and 0.4, with **Haiku 4.5** running the programs and **Opus 5** rewriting them as the reflection model. Haiku is the weakest, cheapest Claude, which is what makes the penalty an interesting tradeoff.

| program | λ | accuracy | LM calls / record | $ / 1k records | mean latency* | 
|---|---|---|---|---|---|
| dspy.Predict baseline | n/a | 90.4% | 1.00 | $0.98 | 1,924 ms | 
| GEPA, prompt-only | n/a | 92.5% | 1.00 | $2.88 | 2,841 ms | 
| Flex + GEPA | 0 | 95.0% | 0.25 | $0.70 | 1,155 ms | 
| Flex + GEPA | 0.05 | 94.6% | 0.17 | $0.45 | 726 ms | 
| Flex + GEPA | 0.1 | 90.8% | 0.07 | $0.18 | 347 ms | 
| Flex + GEPA | 0.2 | 91.7% | 0.08 | $0.09 | 135 ms | 
| Flex + GEPA | 0.4 | 92.1% | 0.004 | $0.01 | 65 ms | 

**Mean per-request latency under 8-way concurrency.*

**Even with calls free (λ=0), the optimizer wrote code.** The metric function scored on accuracy only. The only nudge was in the metric's textual feedback asking for cases to be settled in code where possible. The best program it found routed 75% of records through deterministic Python and came out *more accurate* than calling the model every time at **95.0%** vs 90.4% (McNemar p=0.019), while being **faster and cheaper**. The rules handle the easy cases better than a small model does, and the model only sees the cases that actually need judgment.

**At λ=0.4, the program called the model once across 240 records.** Accuracy held at 92.1%, statistically indistinguishable from the always-call baseline, at roughly a hundredth of the cost and a thirtieth of the latency. The other high penalties land the same way: accuracy at parity with the baseline for a fraction of the price, a trade most production systems would take.

## Reading the Code It Wrote

At λ=0.4, the program holds about two hundred lines of Python code, written by the reflection model. Condensed to its skeleton:

```
class SamePlaceModule(dspy.Module):
    def __init__(self):
        super().__init__()
        # The LLM is a LAST-RESORT fallback: it is only consulted for
        # the narrow band of pairs where the deterministic signals
        # genuinely conflict (e.g. clearly the same brand/name but at
        # a mismatching house number and a middling distance).
        self.judge = dspy.Predict(dspy.Signature(
            "input_name: str, input_address: str, "
            "match_name: str, match_address: str, "
            "distance: float, name_similarity: float, "
            "address_analysis: str -> is_same: bool",
            "You perform entity resolution on business/POI listings. [...] "
            "3. The house number is the strongest address signal. [...] "
            "4. The same brand name far apart means two different "
            "branches => NOT the same place."
        ))
    def forward(self, **inputs):
        import re, difflib
        # ~150 lines of helpers: normalize names (strip '#30696',
        # 'LLC', generic words like CAFE/GRILL), parse addresses into
        # house number + street core, compute fuzzy similarity over
        # the distinctive tokens...
        name_same = (nsim >= 0.87) or (containment and shared_len >= 5)
        name_diff = (not name_same) and (nsim < 0.62)
        decision = None
        if name_same:
            if addr_same and (d is None or d <= 400.0):
                decision = True    # same name, same address, close by
            elif hn_diff and d is not None and d > 120.0:
                decision = False   # different house numbers => branches
            elif d is not None and d > 500.0:
                decision = False   # far apart => different branches
            # ...
        elif name_diff:
            decision = False       # distinctive name parts disagree
        else:
            # gray zone (nsim 0.62–0.87): let the address decide
            if addr_same and (d is None or d <= 150.0):
                decision = True
            elif hn_diff or (d is not None and d > 200.0):
                decision = False
            # ...
        if decision is None:   # rules can't decide -> ask the model
            out = self.judge(**inputs, name_similarity=round(nsim, 3),
                             address_analysis=analysis)
            decision = to_bool(out.is_same)
        return dspy.Prediction(is_same=bool(decision))
```
The comment at the top, "the LLM is a LAST-RESORT fallback," was written by the optimizer about its own architecture. The algorithm underneath it has three stages:

- **Normalize:**Names are uppercased, stripped of franchise numbers ("#30696"), legal suffixes (LLC, INC), punctuation, and some forty generic business words (CAFE, RESTAURANT, MARKET, GRILL...). What remains is the distinctive part of the name: for KIN CAFE, just KIN. Addresses also get parsed into a house number and a street core, discarding unit designators (STE 4, APT B) and street-type words, so that AVE versus AVENUE can never cause a mismatch.
- **Compare:**The distinctive name tokens are scored zero to one with a fuzzy similarity, and binned into three buckets: confident matches, confident misses, and unsure. Addresses are compared piece by piece, with house numbers treated separately from street names, because the reflection model determined that the house number alone is a strong signal in the data.
- **Decide:**Each bucket gets its own rules combining the name verdict, the address data, and the distance between the two geocoded points. For example, if the names and address match and they're within 400 meters: same place. If they're the same name but different house numbers more than 120 meters apart: two branches of one brand.

Only when *none* of the "Decide" rules fire, for example the same brand name at a mismatched house number and a middling distance, does the record go to the model. And it doesn't go alone: the module forwards its own analysis as extra input fields (the parsed house numbers, street cores, and similarity scores) and the judge's instructions distill what it learned into a handful of numbered domain rules. At λ=0.4, this fallback fired once in 240 records.

## Deterministic or Stochastic? Let the Metric Decide

Everyone building with AI keeps re-answering the same question: what do you give to code and what do you give to the model? It's the same instinct you see in coding agents, which increasingly write a Python script to do a job rather than doing it token by token.

With Flex, we can let a model explore this space as it gets feedback from the student model and the metric. It can get quite sophisticated. For example, we pointed Flex and GEPA at SWE-bench Pro, a coding benchmark comprised of Github Issues. Given the high-level task of reading an issue and generating a fix, and a few tools, Haiku 4.5 resolved 0 out of 12 sampled issues.

We then ran GEPA on our Flex program, with `max_metric_calls` set to a modest 60. This optimized program resolved 4 out of 12 problems, after designing a software engineering workflow that mixed Python and LLM calls to research, draft, evaluate, repair, and submit a final answer. Its self-written instructions included guards against specific failure modes.

This experiment is a pilot and should be read as one. But it is striking to watch a harness evolve in a handful of turns to 4 of 12, against the 39% Haiku is reported to reach inside a mature, hand-built harness.

Today, most of our harnesses are written once and edited as issues arise. With Flex, we can continually compile our harness, balancing what's written as code and what's handed off to an LM, as new data, models, and tactics arrive. As product builders, we can sweep models and tweak our metrics to find the optimal balance between cost, latency, and accuracy for our applications.

As we've watched GEPA rewrite programs across many types of tasks, four moves keep showing up:

- **Decomposition.**Noticing that a task has steps (parse, normalize, compare, decide) and giving each step its own implementation.
- **Method selection.**Choosing, for each step, between deterministic code and a model call, and picking the right module (Predict, ChainOfThought, RLM) when it's a call.
- **Routing.**Recognizing that different- *inputs*are different tasks: clear cases down the cheap path, ambiguous ones to the judge.
- **Evolution.**Once the structure settles, refining what's inside it: the signatures and instructions of the decomposed parts, and the code itself.

Hand-written harnesses make these moves too. But they don't evolve with new models, datasets, or tactics unless you specifically go back and rewrite them. The models are good enough at programming that we can now continually compile our harnesses as we get more data, better metrics, and better models.

## Where This Comes From

Flex builds on a few research threads:

- **GEPA.**Lakshya Agrawal and team's reflective prompt evolution, from the Sky Computing Lab at UC Berkeley. It showed that a model reading execution traces and metric feedback can out-optimize reinforcement learning on rollout efficiency.
- **Meta-Harness.**Yoonho Lee and team's work out of Stanford on treating the harness around a model as a learnable object.
- **RLM.**Alex Zhang and team's work on Recursive Language Models at MIT, where a model explores a large input programmatically instead of swallowing it whole. RLM is one of the primitives Flex's optimizer can reach for when a step needs it.
