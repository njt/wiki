---
url: https://sgnt.ai/p/jev/
title: "You could have built Jev"
author: not stated
date_fetched: 2026-09-19
date_published: not stated (self-dated to Sep 2026: "probably massively out of date if you're reading it any time after Sep 2026")
topics:
  - ai-research-and-models
  - local-and-open-source-inference
---

A first-principles teardown of Jev, TypeSafe's classification model that promises "NO HALLUCINATIONS" and answers with a single token plus a probability. The author's thesis is that there is nothing mysterious to understand: an LLM already emits a probability for every token in its fixed dictionary on every forward pass, so a Jev-like is just a prompt that names the answer options as characters ("0"/"1"), a read of the logits for those specific token IDs, and a softmax restricted to those two entries. One forward pass, no generated text, no visible reasoning — and "no hallucinations" turns out to mean *answer-shaped*, not *correct*.

The article walks through building one by hand: a constant, prompt-cacheable prefix compiled from the question schema (TypeSafe's sandwich-classifier JSON is the worked example), the dynamic state pushed in as a suffix, a blunt instruction to output one character, and a parser that encodes "0" and "1" with the model's own tokenizer, asserts each is a single token, and pulls the two logits out of the next-token array. A worked softmax example (logits 1.2 vs 3.2 → 11.9% vs 88.1%) settles the ice-cream-sandwich question. The author notes the one real annoyance: models like to chat, so text-in/text-out parsing is unreliable — which is precisely why reading logits instead of sampled text is the honest version of the trick.

The competitive landscape materialized faster than the article could be written: four projects shipped while it was being drafted. TheoLeeCJ/openjev applies the exact technique with Qwen3.5-4B and reconstructs 84.5% agreement with TypeSafe's reference answers (Jev got 88.3% on the same subset), with a 5× speedup from returning a single token. ekzhang/openjev-sglang runs 1,000 MMLU-Pro questions against live Jev (83%) versus Qwen3.6-35B-A3B (59%) and Qwen3.8-27B (60%) — a large gap — but only a small gap on 3,270 BoolQ questions (89% vs Jev's 91.56%). S Anand's OpenRouter comparison finds Jev slightly faster and cheaper than DeepSeek v4.1 Flash and GPT-5.6 Luna at a slight accuracy cost, on 77 tiny samples the author is skeptical of. TypeSafe's own adaptor, benchmarked against closed models without logit access, just asks the model nicely for a choice string and retries — which nobody involved considers a fair test.

TypeSafe's claimed advantages are a secret architecture, a new training method (Reinforcement Learning for Calibrated Decisions, RLCD), and "parallel sampling" (answering several questions in one sweep). The author's verdict is suspended: the technique is "not all that clever, original, or secret," and the decisive experiment — running an open clone in front of a frontier model — has not been run, because frontier APIs don't expose logits. Either Jev's advantage largely evaporates, or the architecture and training differences are a durable optimization. The closing promise: "You should now be able to reason about Jev."
