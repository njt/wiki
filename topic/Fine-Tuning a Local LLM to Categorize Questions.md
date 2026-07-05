# Fine-Tuning a Local LLM to Categorize Questions

Torgeir Helgevold's clean, numbers-driven experiment in making a 600M-parameter model reliable enough for production classification. The progression from 10% to 79% to 92% accuracy is less interesting than *how* he got there: semantic overlap is the enemy of small-model classification, and the fix isn't better training — it's removing semantics from the output vocabulary entirely.

---

## Key Quotes

> "Can a tiny local model be finetuned into a reliable classifier of household questions?"

The framing matters. Not "can a tiny model do this" — prompt alone gets 10%. The question is whether fine-tuning crosses the reliability threshold for production use. At 92% the answer is "usable predictor, not flawless classifier."

> "The default fine tuning parameters provided by Unsloth provide a very good starting point."

This is the pragmatic line that separates tinkerers from practitioners. Helgevold explicitly chooses not to hyperparameter-tune because the data quality lever is bigger. This mirrors the broader LLM trend — architecture and hyperparameters are commoditizing; dataset curation is where the work lives.

> "Return only the short label code from the list. Never return the category name, a number, a synonym, an explanation, or any other text."

The two-character opaque ID is the article's sharpest insight. When categories have semantic overlap (pool vs. water heater vs. fountain — all "watery"), a model will drift toward the semantic neighborhood of the right answer. By replacing "hvac" with "KK" and "pool" with "OO," you eliminate the drift surface. This is the [[Tone LLM]] contract/adapter pattern in miniature: the LLM outputs a constrained symbol, deterministic code maps it to meaning. The model's job shrinks to "pick the least-wrong token" and a two-letter code has far fewer wrong tokens near it than a semantically loaded word.

> "Water-based confusion from fountain, water heater and pool."

Even after the opaque-ID trick, the remaining 8% error is dominated by one confusion. This is genuinely hard — it's not a format problem but a semantic proximity problem in the training data. More examples of water-adjacent categories with distinguishing features would be the next lever, not prompt engineering.

---

## Key Themes

- **#pattern** — Opaque output encoding as defense against semantic drift. Same principle as [[Tone LLM]]'s JSON schema constraint but lighter: arbitrary two-char codes instead of structured output APIs.
- **#tool** — Unsloth + QLoRA as the default fine-tuning stack for consumer hardware. The author's "don't touch the defaults" stance is a data point for Unsloth's maturity.
- **#concept** — Fine-tuning as reliability engineering. The goal isn't to make the model smarter — it's to make its failures predictable and its outputs constrained enough for a downstream pipeline to consume.
- **#tool** — Qwen 3:0.6B as a classifier substrate. 600M parameters is tiny by 2026 standards. The fact that it reaches 92% on a multiclass task with <1K training examples suggests fine-tuning small models for narrow classification is deeply under-explored relative to the "prompt a big model" default.

---

## Critical Analysis

**The opaque-ID trick is the keeper.** Everything else in the article is standard fine-tuning practice (Unsloth, QLoRA, 70/15/15 split). But the insight that semantic overlap in category names *is* the failure mode, and that the fix is removing semantics from the output space rather than adding it to the training data, is genuinely clever. It's the kind of pattern that should be in every practitioner's toolkit alongside structured output and constrained decoding. The model can't hallucinate "near" a category if "near" has no meaning.

**The article underplays its own best lesson.** Helgevold frames the second attempt as "a minor change to the prompt" but it's actually a fundamental redesign of the classification contract. The model's job went from "understand the category and output its name" to "match the pattern and output a symbol." That's a different task — easier for a small model to get right, harder to audit when it goes wrong (a "KK" misclassification tells you nothing about *why* the model was confused, unlike "hvac" vs "air conditioning").

**92% is impressive but the failure mode is instructive.** The remaining errors cluster around one semantic domain (water-adjacent concepts). This is *good* — the system fails predictably. You know where to add training data. A system that fails randomly across all categories at 92% would be harder to improve. Helgevold's plan to revisit the training data for water categories is exactly right, and the fact that he can predict *which* categories will fail tells you the approach is sound.

**The 600M-parameter question.** At 0.6B parameters, this model runs on basically anything. The article doesn't dwell on hardware but the implication is clear: fine-tuned tiny models can do production classification work that would otherwise consume a cloud API call. For latency-sensitive or air-gapped applications, this matters. The household chatbot framing is almost too humble — the pattern generalizes to any system that needs reliable, fast, local classification as a preprocessing step before more expensive LLM calls.

**What's missing:** The article doesn't benchmark against just using the 4B model with a good prompt. At what point does the larger model's zero-shot classification beat the fine-tuned 0.6B? The 4B with the opaque-ID prompt might get 95%+ without fine-tuning at all. The economics (fine-tuning cost vs. incremental inference cost) would be useful. Also missing: latency numbers. Classification speed matters in a chatbot where the category tag needs to be assigned before the RAG lookup and answer generation can start.

---

## Related Pages

- [[Your Coding Agent Should Do AI System Engineering]] — Burtenshaw on coding agents doing fine-tuning; Helgevold's experiment is exactly the kind of AI systems engineering he argues agents should handle
- [[FLUX.2 klein LoRA Fine-Tuning]] — LoRA fine-tuning on a single consumer GPU; same QLoRA approach, same "defaults are fine" pragmatism
- [[Honey I Shrunk the Coding Agent]] — Scaffold redesign for small Qwen models; Helgevold's opaque-ID prompt is a scaffold-level fix, not a model fix
- [[Local Models in Mid-2026]] — The model landscape Helgevold is working within; Qwen 3 family, MoE architectures
- [[Self-Hosted LLMs]] — Running models locally; the hardware economics that make a 0.6B classifier viable
- [[AI Engineering for Developers]] — Cavallin's adaptation spectrum (prompt → RAG → fine-tuning → agents); Helgevold makes the case for fine-tuning as the right tool when classification reliability matters
- [[Tone LLM]] — The contract/adapter pattern: LLM fills a constrained output schema, deterministic code handles the rest. Helgevold's opaque IDs are the same pattern at smaller scale
- [[Smart Models Dumb Pipes]] — LLMs as judgment machines, not Q&A machines. Classification is judgment; the two-char code output makes it a dumb pipe downstream
- [[A Non-Anthropomorphized View of LLMs]] — LLMs as functions through ℝⁿ. Fine-tuning for classification is a particularly clean case: you're optimizing a function to map questions to categories, nothing more

---
*Sources: [[summary/fine-tuning-local-llm-categorize-questions]]*
*Last updated: 2026-06-22*
