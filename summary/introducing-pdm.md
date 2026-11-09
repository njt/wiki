---
url: https://www.flashmind.com/en/research/introducing-pdm
title: "Introducing PDM"
author: Flashmind
date_fetched: 2026-11-09
date_published: 2026-11 (benchmarks as of 2026-11-01)
topics:
  - ai-research-and-models
  - ai-product-and-business
---

Flashmind, a Dubai-based company spun out of the trading firm Infinite Field, launches PDM — a family of three zero-shot time-series forecasting foundation models: PDM-Nano (119M), PDM-Pico (32.2M) and PDM-Femto (6.18M) parameters. They predict future values from past measurements with no per-series training, and all three share the same custom architecture and capabilities, differing only in capacity.

The headline claim: as of 1 November 2026, PDM-Nano ranks #1 among zero-shot foundation models by normalized MASE on both GIFT-Eval (0.637) and TIME (0.625), with results independently reproduced and merged by the benchmark maintainers across 97 GIFT-Eval configurations, 98 TIME configurations and 100 fev-bench tasks. Accuracy improves monotonically with size, and small models punch far above their weight: 32.2M Pico beats Mistral's Timestral-209M with 6.5× fewer parameters, and 6.18M Femto beats Google's TimesFM-2.5 (37× larger). Nano also beats Datadog's Toto-2.0-2.5B on normalized MASE with ~21× fewer parameters, though Timestral keeps a small CRPS edge on TIME and fev-bench.

Beyond raw forecasting, each model supports multivariate forecasting of related series, past and known-future covariates (the café example shows a discount and closure cutting MAE from 29.5 to 18.0 drinks), missing observations, 16,336-step context, a 960-step native horizon, and 21-quantile uncertainty bands. Models are interchangeable via a single API name change; a free account gets 10,000 forecast calls per month, and there is an MCP integration so an AI assistant can drive forecasts. The company frames its Dubai base and colocated training infrastructure as part of a pitch for the UAE as an AI research hub.
