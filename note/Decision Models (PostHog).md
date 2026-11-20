# Decision Models (PostHog)

Jina Yoon's PostHog field report on decision models — small models that return a typed answer with a calibrated probability instead of LLM prose. The article is a production-tested taxonomy: gate expensive LLM calls, route between models, judge face-value inputs, and know exactly where the approach collapses.

---

## What it argues

LLMs "love to yap": free-form, overconfident prose where a probability would do. A decision model instead answers a narrow structured question — *Is this a mammal? → Yes, 0.99* — and, if calibrated, a 0.8 answer is right about 80% of the time. PostHog runs several in production, and each use case teaches the same lesson from a different direction: decision models are cheap, fast, single-pass judges whose value comes from *not* being language models.

The four uses:

1. **Gates.** Two narrow yes/no questions cut PII false flags to a tenth of what one broad question produced, and 98.7% of session summaries skip the LLM entirely. Thresholds encode cost asymmetry: 0.4 for PII scrubbing (a missed name is stored personal data; a false flag just costs a call), 0.85+ for the Slack bot's "should I speak?" check (an unsolicited reply is the expensive mistake).
2. **Routers.** One `choice` question picks the cheapest model that finishes correctly on the first try, with per-option descriptions — "GPT Luna: light and very cheap... good for short answers, lookups, summaries and very small edits."
3. **Face-value checks.** Embeddings produce the candidate list; the decision model only judges each result against the query — a 25% ranking improvement over raw embedding distance.
4. **Overqualified-LLM replacement.** The SQL chart picker, where deterministic rules ("date column + number column + rows → line chart") gave way to a model looking at the query and the first three rows.

The failure catalogue is equally instructive: thresholds don't survive model version bumps (v1→v2 recalibration by hand-labeled data, hence a model-agnostic AI gateway), chained decision-model jobs flooded the model server for hours, and ranking PR-review findings hit AUC 0.52 because Jev can't take the context. The escape hatch is the hybrid Jeeves, which "reasons before deciding when it's unsure."

## Key quotes

> "A probability score of 0.85 still means ~150 wrong answers per 1,000 checks, so plan ahead for false positives and false negatives."

The whole discipline is here in one sentence. Decision models don't eliminate error; they make error *priceable* — you pick the threshold that prices the two error types correctly, and the 0.4-vs-0.85 contrast shows that "correct" flips entirely with the cost structure.

> "Decision models judge what's in front of them in a single pass. Leave multi-step reasoning to LLMs."

The one-sentence boundary of the technique. PostHog's Highlights-feed experiment — two chained questions, 32 sessions and 12–38k tokens per request — is the cautionary tale: the failure wasn't accuracy, it was operational (an hourly job blocking other products' calls for hours).

> "Use decision models as evaluators for inputs that are too simple for a language model, yet too fuzzy for a regex."

This slots decision models into the gap between deterministic tooling and LLMs — exactly the gap most teams currently bridge by sending everything to a frontier model and paying for it.

## Themes

#concept #pattern #tool

## Analysis

This is one of the more useful pieces of applied AI engineering writing because it does something rare: it names the *boundary* of a technique instead of only the wins. The 98.7% gate-skip figure is the headline, but the honest material is the AUC 0.52 and the server-blocking chained job — the places where the cheap-tool temptation broke.

The deepest idea is that calibration converts model output into engineering input. Once an answer carries a probability, you can write policy against it: `if score ≥ 0.4: escalate`. That's a category shift — the model becomes a component in a control flow, not a conversation partner. It also explains why thresholds matter so much more than raw accuracy: a miscalibrated 0.85 silently breaks every downstream policy tuned to it, which is exactly what bit them when the model version changed.

The routing use case deserves more attention than it gets. Most teams pick one model per feature at design time; a `choice` question with per-option notes makes model selection a runtime, per-request decision — the same instinct behind efficiency-frontier routing elsewhere, but implemented in 100 lines rather than infrastructure. The caveat that thresholds must be per-task-per-model, recalibrated against hand labels, is the honest cost of that flexibility.

Skepticism warranted: this is PostHog writing about PostHog's own Jev and in-house Jeeves, and the "444.6x cheaper" figure is doing marketing work. The patterns, however, transfer regardless of whose decision model you buy — and the piece's own coda ("expects this article to be obsolete in two months") is fair; hybrid reason-then-decide models like Jeeves will likely absorb use case 3 within the year.

## Connections

This strengthens [[Deterministic When Possible, Probabilistic When Necessary, Human When Cheaper]] by naming the missing middle tier: that piece's "probabilistic when necessary" is exactly this article's "too fuzzy for a regex, too simple for an LLM" band, here with a concrete mechanism and threshold calculus. It nuances [[Managing AI Coding Costs at Scale]] — Databricks's "intelligent routing" as infrastructure becomes a single `choice` question in PostHog's hands, suggesting routing belongs in the harness before it belongs in a gateway. And it complements [[Reliability Is Outside the Model (O'Reilly)]]: PostHog's per-model, per-task threshold configuration is a working instance of that piece's claim that reliability comes from the system around the model, not the model itself. It also complicate's [[A Platform Engineering Playbook for Production LLMs]]: the model-agnostic AI gateway wasn't platform-first grand strategy — it was forced into existence by a threshold drift bug.

---
*Sources: [[raw/decision-models]], [[summary/decision-models]]*
*Last updated: 2026-11-20*
