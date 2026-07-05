---
url: "https://epoch.ai/data-insights/open-closed-eci-gap"
title: "Open models lag state-of-the-art closed models by 4 months"
author: "Jack Edwards and Luke Emberson"
date_fetched: 2026-06-05
date_published: 2026-05-29
source: "Epoch AI"
license: "CC BY 4.0"
type: "data-insight"
slug: "open-closed-eci-gap"
tags:
  - open-weight-models
  - closed-models
  - AI-benchmarks
  - model-capability
  - ECI
---

# Open models lag state-of-the-art closed models by 4 months

**Authors:** Jack Edwards and Luke Emberson
**Published:** May 29, 2026
**Organization:** Epoch AI
**License:** Creative Commons Attribution 4.0 (CC BY)

## Core Finding

From January through May 2026, leading open-weight models trailed frontier closed models by approximately four months on average, as measured by the Epoch Capabilities Index (ECI).

The ~8-point ECI gap is described as similar to the difference between GPT-5 and GPT-5.5.

## Key Data Points

- **Analysis window:** January 1, 2026 – May 28, 2026
- **Average time lag:** 4 months
- **Average ECI gap:** 8 points, with a 90% confidence interval of 7 to 11 units
- **Prior period (Jan 2023 – Oct 2025):** ~3 months average lag (from their October 2025 Data Insight)
- **Trend:** Gap widened slightly

## Methodology

Day-by-day analysis over the window:

1. For each day, identify the best open-weight model by ECI score available as of that date
2. Compare against the historical SOTA ECI frontier
3. Determine the most recent date on which the top closed model was "not significantly better" than the open-weight model
4. Time gap = days elapsed since that date

**Bootstrap approach:** Because ECI scores carry uncertainty, bootstrap sampling is used. For each bootstrap, the open-weight model's ECI estimate is compared against each historical SOTA model's bootstrapped ECI estimate (preserving pairing across samples).

**Significance threshold:** An open-weight model is considered to have plausibly caught up to a prior SOTA if it "outperforms that SOTA model in at least 5% of paired bootstrap samples."

**Alternative threshold:** If the bar were raised so the open-weight model's point estimate must strictly exceed the closed model's (rather than using the 5% bootstrap criterion), the average gap would grow to **six months**.

## Limitations (Authors' Own)

1. **Public benchmark overfitting:** Open-weight models "tend to perform worse on private benchmarks compared to closed models" because they may more aggressively optimize against public benchmarks. This suggests the true gap is *larger* than measured.

2. **Selection bias in coverage:** Only models with sufficient public benchmark data to assign an ECI score are included. Leading closed labs sometimes withhold their most capable models "for safety, commercial, or competitive reasons." This also biases the gap estimate *downward*.

## Related Resources Cited

- Prior insight: "Open-weight models lag state-of-the-art by around 3 months on average" (Oct. 30, 2025)
- arXiv: 2405.00332v1
- LessWrong post on how far behind open models are

## Citation

Jack Edwards and Luke Emberson (2026). Published online at epoch.ai, retrieved from `https://epoch.ai/data-insights/open-closed-eci-gap`. Accessed June 5, 2026.
