# Goodhart's Law and AI Benchmarks

When a measure becomes a target, it ceases to be a good measure. Charles Goodhart's 1975 insight about British monetary policy now governs the benchmarks that decide AI funding, procurement, and press coverage — and the evidence has crossed from anecdote to quantitative fact.

---

## Key Quotes

> "The mechanism designed to catch cheating did not survive contact with a Web-scale crawler."

Williams opens with the BIG-bench canary story: researchers embedded a unique string so anyone training a model could filter the benchmark out of their corpus. OpenAI's GPT-4 contamination checks found the canary had been swallowed into the training data anyway. The model can reproduce it on request. This is the article's thesis in miniature: every defense against benchmark gaming eventually fails, because the incentive to hit the number is stronger than any mechanism designed to protect it.

> "A public benchmark score tells you what a student's exam result tells you when the student had the paper in advance."

The GSM1k finding distilled to one sentence. The correlation between verbatim GSM8K reproduction and score gaps (Spearman's r² = 0.36) is the quantitative evidence that makes this more than a suspicion. This isn't about cheating — nobody decided to cheat. The pipeline cheats by default, because benchmarks live on the public web and web-scale pretraining hoovers all of it up.

> "We have been ranking billion-dollar systems, to a decimal place, on answer keys we never proofread."

Gema et al. found 6.49% of MMLU questions contain errors. In the virology subset, 57% of analyzed questions were flawed. Truong et al.'s "Fantastic Bugs" audit found up to 84% of top flagged questions had substantive flaws across nine benchmarks. GSM8K itself carries a ~5% error rate. The decimal-place precision of leaderboard rankings is built on error-riddled answer keys — and a model can climb the leaderboard by learning the mistakes.

> "No villains rigged a leaderboard here. Incentives quietly corrupted a well-intentioned academic project, which is exactly what Goodhart predicts."

The Chatbot Arena story is the article's most important structural argument. The Arena was the community's answer to contamination: live human preference instead of static test sets. It worked — until providers started using best-of-N private testing to select which model variant to publish. Meta tested 27 private Llama-4 variants and published only what it chose. Two identical checkpoints scored 17 points apart. Nobody cheated. The incentives did the work.

> "The question is how to make decisions knowing that every benchmark you trust is already partly fiction."

The constructive thesis. Not "how to build an ungameable benchmark" — 50 years of evidence from monetary policy to standardized testing says there is no such thing — but "how to act given that gaming is inevitable."

## Key Themes

- **#concept Goodhart's Law in AI**: The foundational dynamic — when benchmark scores decide funding, they become targets. The four variants from Manheim and Garrabrant (2018): regressional Goodharting (test becomes part of training) and adversarial Goodharting (evaluation process gets gamed) are the two dominating the current crisis. This isn't a bug awaiting a patch; it's a structural property of any system where metrics carry high stakes.

- **#concept Contamination as Default**: The most important shift in thinking Williams advocates: contamination isn't something bad actors do; it's the default state of any public benchmark in the web-scraping era. A "held-out" test set that's been public for three years is not held out. Nobody has to decide to cheat; the pipeline cheats by default. GSM1k is the template for the only defense that works: private, freshly-commissioned test sets.

- **#pattern Benchmarketing**: The word captures the fusion of benchmarking and marketing that now defines model launches. Providers select temperature, prompting, and few-shot configuration to maximize the headline number — each choice defensible in isolation, the sum describing a model nobody will ever run. The incentive chain (leaderboard → press → fundraising → procurement) means nobody in the chain is paid to ask what the number means. The consumer-grade version is the YouTube "benchmark," where the measured quantity is `pp`/`tg` on budget hardware with default settings — an input-side vanity metric, not the wall-clock-to-correct-result that actually matters for agentic coding ([[Wall-Clock Time and the Qwen3.8-27B Daily Driver]]).

- **#concept Saturation and the Treadmill**: MMLU, HumanEval, and HellaSwag are effectively ceilinged. When every model passes at ~90%, the field moves to a new benchmark, which becomes a target, which contaminates and saturates in turn. Each cycle is shorter than the last. This is a structural driver of benchmark inflation — it's not that models are getting better at the same rate, it's that we keep moving the goalposts to new, uncontaminated tests.

- **#concept Private Evaluation as the Only Honest Answer**: The article's most practical recommendation: a 20-50 question mini-benchmark built from your actual tickets, contracts, or queries beats any public leaderboard for predicting how a model performs on your work, because nobody has trained on your workload yet. This is the same thesis as [[LLM Evals]] ("build custom evals from real failures") and [[Dan Luu on AI Coding]] ("run your own benchmarks"). If you pair it with an LLM judge, control the judge's known biases toward position, verbosity, and its own outputs — or you'll Goodhart your own private eval.

- **#pattern The Leaderboard Illusion**: The Singh et al. (2025) findings are the most thorough empirical documentation of leaderboard gaming to date. The structural fixes are written down (prohibit score retraction, require disclosure of every variant tested, cap private submissions, equalize sampling), but the article is skeptical any leaderboard can hold that line while its biggest users are also its biggest names.

## Critical Analysis

**The article's greatest strength is its historical framing.** Williams situates AI benchmark gaming in a 50-year lineage — Goodhart (1975), Campbell's Law, No Child Left Behind, BLEU in machine translation — which makes the argument harder to dismiss as "just this model" or "just this benchmark." Computing keeps rediscovering and forgetting this pattern. The article's contribution is documenting that we've crossed from anecdote to quantitative evidence.

**The Chatbot Arena analysis is the freshest contribution.** The identical-checkpoint divergence (17 points), the best-of-27 selection effect, and the data asymmetry (top two proprietary providers get 19-20% of battle data each, 83 open-weight models combined get under 30%) are all new findings that hadn't been publicly documented at this level of rigor. The article gives LMArena's rebuttal fair space — overlapping confidence intervals, public testing policy since March 2024 — but correctly notes that none of the rebuttals dispute the structural problem: well-resourced labs systematically use best-of-N while smaller labs submit once, and the ranking model assumes nobody does.

**The constructive section is honest about costs.** Private test sets "rot slower, and slower is what we can actually buy." Human-written expert-verified questions are expensive. Contamination detection lags gaming and likely always will. These are not silver bullets; they're triage. The honesty is more useful than a false promise of a fix.

**What's missing: the ecosystem question.** The article focuses on individual evaluation integrity but doesn't address whether the ecosystem of benchmark producers, model labs, and enterprise buyers can self-correct. If every lab's incentive is to game, and every buyer's incentive is to use public benchmarks (because private evals are expensive), what breaks the equilibrium? The article's answer — "evaluate on your own data" — is correct but shifts the cost to the buyer, who is also the party with the least information about how to evaluate well. A related pattern the article doesn't name: selective publishing of benchmark tiers. DeepSeek V4 Flash 0731 reports ARC-AGI-1 (89.0%) and ARC-AGI-2 (61.4%) but leaves ARC-AGI-3 blank — not because the harder tier is irrelevant, but because missing scores are a marketing decision as much as present ones ([[DeepSeek V4 Flash 0731 ARC-AGI Results]]).

**The canary story as framing device is elegant but incomplete.** The BIG-bench canary was contaminated not because someone cheated but because web-scale crawlers can't realistically filter every canary string from every dataset. This is a different failure mode than the best-of-N gaming on Chatbot Arena, and the article collapses them into one narrative. They're both Goodharting, but they require different defenses: canary contamination needs private test sets; best-of-N gaming needs submission transparency rules. Conflating them risks recommending the wrong fix.

**The article sits at the intersection of several threads in this wiki.** It extends [[Benchmark Exploitation]]'s adversarial perspective (how benchmarks can be gamed) into the structural question (why they inevitably will be, regardless of defenses). It provides the quantitative evidence that [[Dan Luu on AI Coding]]'s Indurain analogy and "basically meaningless" claim were pointing at. It reinforces [[LLM Evals]]'s core advice to build custom evals from real failures. And it complicates [[FrontierCode]]'s ambition to build a better benchmark: if Goodhart's Law is structural, a better benchmark just becomes a better target.

---

## See Also

- [[Benchmark Exploitation]] — The complementary adversarial perspective: how specific benchmarks can be exploited, including the seven vulnerability patterns
- [[Dan Luu on AI Coding]] — The Indurain analogy and the case that single-number benchmarks are "basically meaningless"
- [[LLM Evals]] — Hamel Husain's guide to building custom evals from real failures, the constructive answer
- [[FrontierCode]] — Cognition's attempt at a better benchmark (mergeability over correctness); the question is whether it's Goodhart-proof
- [[Demystifying Evals for AI Agents]] — Anthropic's framework: grade outcomes, not pathways
- [[Text-to-SQL in the Real World]] — The same pattern in a different domain: public benchmarks at 90%+ accuracy, real enterprise data warehouses humble LLMs to 10%
- [[How Far Behind Are Open Models]] — Contamination audit as methodology; the conservative estimate that open models trail closed by 8-10 months
- [[Five Studies That Are Changing How I Think About AI in Software Engineering]] — The broader story: AI accelerates upstream coding, everything downstream breaks

---
*Sources: [[raw/goodharts-law-comes-for-every-benchmark-you-trust]], [[summary/goodharts-law-comes-for-every-benchmark-you-trust]]*
*Last updated: 2026-08-06*
