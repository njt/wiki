# Microsoft-Decision-1 and the Foundry Decision Play

Microsoft enters the decision-model market with Microsoft-Decision-1 — a Qwen3.5-9B post-trained model that returns calibrated probabilities over a fixed answer set instead of text — launched in Foundry at $0.042/M input tokens with output free, benchmarked first on accuracy and fastest on latency against the existing Jev-class field, and pitched as the cheap, fast judgment layer that lets agents "guide and control" themselves.

---

Microsoft-Decision-1 is the largest vendor's answer to a category the wiki has been tracking through TypeSafe's Jev, Marc Brooker's Hobson, and the surrounding ecosystem. The pitch is exactly the System One thesis: LLMs generate text and reason; decision models "deliver structured outputs that software can immediately act on" — decisions and classifications at very low cost with high performance. The product form is a structured API call that takes a fixed set of answer options and returns a calibrated probability for each, supporting yes/no, multiple-choice, ratings, and rubric-based grading of AI responses and agent actions. Notably, Microsoft post-trained on Qwen3.5-9B — an open-weight torso, same recipe class as [[Hobson — Brooker's Home-Built Jev-Class Classifier]] — and says it will rebase on MAI and OpenAI models, signaling that decision models are a *head* problem, swappable across torsos.

---

## Key quotes

> "Unlike LLMs, which are designed to generate text or reason through complex problems, decision models are purpose-built to deliver structured outputs that software can immediately act on."

The category definition, verbatim from a company with a distribution engine Foundry-size. When Microsoft canonizes a model class, the question stops being "is this real" and becomes "what does the incumbent's entry do to pricing and differentiation" — and the answer here is aggressive: $0.042/M input, output free.

> "Adding just 100 milliseconds to each of 20 sequential decisions adds two seconds to the overall workflow."

The best engineering argument in the post, and the one that matters for agent harnesses: in sequential decision loops, latency compounds. This is why the p50 latency framing (85 ms vs GPT-6 Sol's 3,010 ms) is not just benchmark bravado — a 35× latency win inside a 20-step control loop is the difference between an agent that feels instant and one that doesn't.

> "Applications use confidence to decide when to act, defer, or ask for review, so a 90% prediction should be right about nine times out of 10 on representative cases."

Calibration as an API contract, not a metric. This is the sentence that connects the whole post to [[Jev Can't Be Calibrated]] and the Brooker critique: confidence-gated act/defer/hand-off routing only works if the numbers mean what they say. Microsoft claims second place, 0.9 behind Quyet-1.0-Large — a genuinely competitive framing, since every model here scores within 17 points of perfect on their axis.

> "Microsoft Discovery implements an adaptive replanning feature where an agent evaluates a previous experiment, revises its approach based on rubric grades, and repeats until it has completed its objectives."

Rubric-graded replanning loops inside Microsoft's own research product — the same grade-and-repeat pattern the wiki has seen in agent verification workflows, now running on a dedicated scoring model rather than the reasoning model itself.

## Key themes

- #concept — **Decision models as a tier below LLMs**: structured outputs over fixed option sets, not generation. Microsoft's framing essay explicitly positions cost as the driver: "cost plays a major role in how people decide to use AI… choose the right model for the right job."
- #pattern — **Confidence-gated control**: the probability is "part of the API," consumed by applications to act, defer, or escalate to human review — the act/confirm/hand-off routing that [[Building with Jev Skill]] codified as a Claude Code skill, now shipped as a first-class hosted model.
- #tool — **Foundry distribution**: the interesting competitive fact is less the model than the channel — Microsoft putting a decision model inside the platform where enterprise agents already live, plus OpenRouter for the rest.

## Opinionated analysis

The launch is competent vendor marketing with unusually good bones. The eval design is better than the genre norm: 36 benchmarks "kept blind from training," perturbation robustness measured explicitly (1.3% flip rate, zero flips on paraphrase/shuffle), calibration scored against rivals with Microsoft's own model losing that category. A company willing to print "second on calibration, 0.9 behind Quyet" is not running the usual benchmark shell game.

But the standard Jev-class caveat applies: the benchmarks are Microsoft-selected, the latency measured "through Foundry in the same region" (their model's home turf), and the scale-cost chart is Microsoft-estimated with identical input assumptions — the $11 vs $2,434 per million texts figure is a marketing chart, not an invoice. And the deep problem identified in [[Jev Can't Be Calibrated]] — that calibration claims rest on self-scored hold-outs — is untouched here; "blind from training" is claimed, not audited.

The strategically interesting move is what this does to the market rather than the model. Microsoft entering validates the category thesis in [[System One Models and Jev]] and [[Which Decision Model Should You Use]], while its near-free pricing does to the small decision-model vendors what frontier-subsidized LLM pricing did to mid-tier labs. The open question this source cannot answer: whether the incumbent's decision model is actually calibrated where it counts, or whether "second, 0.9 behind" is the new "close enough" that erodes the confidence-gating contract the whole category depends on.

## Related pages

- [[System One Models and Jev]] — the foundational case for this model class; Microsoft's launch strengthens it decisively by putting the weight of Foundry behind the category definition, while re-framing it from a startup niche to a platform feature.
- [[Hobson — Brooker's Home-Built Jev-Class Classifier]] — Brooker's independent replication used the same Qwen-torso-plus-head recipe Microsoft now productizes; this source strengthens his thesis that the moat is not the architecture, and complicates it, since Microsoft's claims rest on the same unverifiable hold-out structure he admitted to.
- [[Jev Can't Be Calibrated]] — Microsoft's calibration-as-API-contract pitch is the direct answer to this critique; whether a 92.2 calibration score from vendor-selected benchmarks actually satisfies the act/defer/hand-off contract remains exactly the question that page raises.
- [[Which Decision Model Should You Use]] — the field's comparison page gains its first hyperscaler entry; the pricing structure (output free) and rebase roadmap (MAI/OpenAI torsos) change the practical calculus that page tracks.

---
*Sources: [[raw/microsoft-decision-1-model-foundry]], [[summary/microsoft-decision-1-model-foundry]]*
*Last updated: 2026-10-10*
