# PDM — Flashmind's Time-Series Forecasting Foundation Models

Flashmind (Dubai, spun out of trading firm Infinite Field) launches PDM-Nano/Pico/Femto, a three-size family of zero-shot forecasting foundation models that claims #1 on GIFT-Eval and TIME by normalized MASE — and, more interestingly, claims that its 6M-parameter runt beats rivals 37× its size.

---

## What the post argues

PDM is a "forecasting foundation model": it learns patterns across many time series during training, then forecasts a new series with no per-series training. The family spans three sizes — Femto 6.18M, Pico 32.2M, Nano 119M parameters — with identical capabilities at every size: multivariate forecasting, past and known-future covariates, missing observations, 16,336-step context, a 960-step native horizon, and 21 quantiles. The launch post leans on an extended worked example (a café forecasting iced-coffee orders with a planned discount and closure) to demonstrate covariates cutting MAE from 29.50 to 18.00 drinks.

The benchmark claims are unusually well-hedged for a launch post: the maintainers of GIFT-Eval and TIME independently reproduced the results and merged the submissions including reproduction code; the evaluation covers 97 + 98 + 100 configurations; and the fine print concedes where PDM loses (Timestral keeps a small CRPS edge on TIME and fev-bench) and that Nano sits second overall on TIME behind TimeRSI-1.5, a system that wraps frozen foundation weights in task-specific programs and backtests.

## Key quotes

> **6.18M-parameter Femto beats TimesFM-2.5, a model 37× its size.**

The whole post's thesis in one line. Like [[Timestral]] and [[TimesFM]] before it, this is more evidence that time-series forecasting rewards domain-specific architectures over scale — the accuracy–size frontier chart is the real product here, not the #1 rank.

> In trading, forecast errors affect decisions made with real money, and understanding uncertainty matters as much as estimating what comes next.

The provenance story explains the design: 21-quantile outputs and calibrated bands are first-class citizens, not afterthoughts. A firm that grew out of HFT treats the distribution, not the point estimate, as the deliverable.

> Our evaluations cover 97 GIFT-Eval configurations, 98 TIME configurations and 100 fev-bench tasks… Both benchmarks' maintainers independently reproduced our results for all three models and merged our submissions, including reproduction code.

Third-party reproduction of a vendor's own leaderboard submission is the strongest thing in the post — and it implicitly positions this against the benchmark-exploitation pattern the wiki tracks elsewhere (see [[Benchmark Exploitation]], [[Goodhart's Law and AI Benchmarks]]).

## Key themes

#concept — zero-shot forecasting: a foundation model for continuous measurements, the time-series analogue of what [[TabFM (Tabular Foundation Model)]] does for tables.
#tool — the `flashpdm` Python library, a free-tier API (10,000 calls/month), and notably an MCP integration so an AI assistant can request forecasts directly.
#pattern — one architecture, three sizes, switchable by changing a model name in the same API call; capability parity across the family.

## Opinionated take

This is the most credible launch-post benchmark section I've read in a while — reproduction code, maintainer verification, explicit concessions in a comparison table where the competitor wins on two cells. That said, the claims are narrow in the ways launch posts are: "normalized MASE among zero-shot foundation models" is a carefully carved leaderboard category (TimeRSI-1.5 beats Nano overall on TIME, once you allow task-specific machinery), and the parameter-count comparisons are ratios, not measured cost or latency on matched hardware. The covariate café example is synthetic and self-reported. Still, the small-model-beats-big-model result, if it holds, matters more than the #1 rank: it says forecasting may be a domain where tiny specialists win, which rhymes with the broader pattern in [[When Smaller Models Win]]. The MCP integration is a quiet but telling detail — forecasting is being repositioned as something an agent calls, not something an analyst opens a notebook for.

## Related pages

- [[Timestral]] — Mistral's time-series family, and PDM's direct head-to-head competitor; this source gives the newest comparative table and nuances Timestral's "two-thirds the parameters" claim with a rival that is now smaller still.
- [[TimesFM]] — Google's forecasting foundation model; PDM-Femto's claimed 37× parameter advantage directly complicates TimesFM's standing as the reference open model in this space.
- [[TabFM (Tabular Foundation Model)]] — the tabular sibling of this trend; together they show foundation models colonising structured numeric data domain by domain.
- [[Why LLMs Fail at Tabular Prediction]] — explains why specialist architectures like PDM beat general LLMs on numeric sequence prediction, the mechanism behind this post's headline numbers.

---
*Sources: [[raw/introducing-pdm]], [[summary/introducing-pdm]]*
*Last updated: 2026-11-09*
