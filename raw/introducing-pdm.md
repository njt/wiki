---
url: https://www.flashmind.com/en/research/introducing-pdm
date_fetched: 2026-11-09
---

Today we’re introducing **PDM**, a family of AI models that predict future values from past data. Built by Flashmind in the UAE, **Nano, Pico and Femto** can forecast demand, energy use, sensor readings and other measurements that change over time. All three are available to try for free and use through the Flashmind API.

PDM works on new data without requiring you to train a model of your own. This is called **zero-shot forecasting**. At launch, PDM-Nano ranks **#1 among zero-shot foundation models by normalized MASE**, a measure of forecast error, on both GIFT-Eval and TIME.1, 2

## From past data to a forecast

Many decisions depend on knowing what is likely to happen next. A shop orders stock before customers arrive. An energy operator plans supply before tomorrow’s demand is known. Forecasts help people make those decisions using the measurements they already have.

A **time series** is simply a sequence of measurements over time: daily sales, hourly electricity use or readings from a machine. You give PDM that history and choose how far ahead to predict, known as the **forecast horizon**. For daily sales, a 14-step forecast predicts the next 14 days.

PDM is a **forecasting foundation model**. It learns patterns across many time series during training, then applies what it has learned to a new series. That is what lets you move directly from supplying data to receiving a forecast, without a separate training process for each shop, building or machine.

## Forecasting with context

Past measurements are only part of the picture. A planned discount, a change in opening hours or a price increase can affect what happens next. PDM can use that additional information alongside the series you want to predict. These extra inputs are called **covariates**.

Consider a café deciding how much stock to prepare for the next two weeks. It plans a 20% discount and will be closed for one day. The chart below compares two PDM-Nano forecasts of its daily iced-coffee orders: one uses sales history alone; the other also uses past temperatures, discounts and opening hours, including the planned promotion and closure.

With the extra information, the forecast anticipates the promotion surge and the drop on the closed day. In this example, the average size of the daily prediction error falls from **29.50 to 18.00 drinks**. This measure is called **mean absolute error**, or MAE.

The shaded band shows uncertainty around the prediction. Its middle line is the **median**, the model’s central estimate; the band spans its 10th to 90th percentiles. These levels are called **quantiles**. All three models predict 21 quantiles, and the API lets you choose which to return. They help you plan for a range of outcomes, such as holding more stock when running out would be costly.

## How we set up the café example

Both forecasts use the same **512 days of sales history** and predict the next **14 days**. The forecast with covariates receives temperature only for past days. Discounts and opening hours include both past values and the café’s known plans for the next 14 days. Future sales are withheld from both forecasts and used only to measure their errors.

Orders are constrained to nonnegative integers. The shaded band belongs to the forecast with covariates. These results come from this synthetic example, rather than a benchmark average. The expandable reproduction script at the end of the article runs both forecasts in one API request.

## Three sizes, the same capabilities

Nano, Pico and Femto share a custom architecture designed for time-series forecasting. They differ in capacity, measured in **parameters**: the values a model learns during training. Nano is the largest; Femto is the smallest.

**119M**parametersThe middle modelPDM Pico

**32.2M**parametersThe smallest PDMPDM Femto

**6.18M**parameters

Accuracy improves with model size on every aggregate metric we report across GIFT-Eval, TIME and fev-bench.

The capabilities stay the same across all three sizes. Alongside forecasting a single series, each model can forecast several related series together, such as sales across stores or readings from connected sensors. This is **multivariate forecasting**. The playground’s linked-sensor example lets you compare it with forecasting a sensor in isolation.

All three also support past and known-future covariates, missing observations, up to **16,336 past observations per series** as context and a **960-step native forecast horizon**. A step follows the interval of your data: one hour for an hourly series, one day for a daily series. You can switch models by changing the model name in the same API call.

## How PDM compares

Benchmarks test models on collections of forecasting problems so their accuracy can be compared. Two key measures are **MASE** (mean absolute scaled error), which measures the accuracy of the central prediction relative to a simple baseline, and **CRPS** (continuous ranked probability score), which evaluates possible outcomes and their predicted probabilities. Lower scores are better for both. On fev-bench, results are shown as skill scores, where higher is better.

As of November 1, 2026, **PDM-Nano ranks #1 among zero-shot foundation models by normalized MASE on both GIFT-Eval and TIME**, with scores of **0.637** and **0.625**, respectively. Both benchmarks’ maintainers independently reproduced our results for all three models and merged our submissions, including reproduction code.6, 7

Our evaluations cover **97 GIFT-Eval configurations, 98 TIME configurations and 100 fev-bench tasks**.6, 7, 8 The charts below show the family’s results across these benchmarks.

The gains extend across model sizes. From Femto to Nano, normalized GIFT-Eval MASE falls by **6.9%** and CRPS by **7.3%**.1 On that benchmark, **32.2M-parameter Pico beats Timestral-209M with 6.5× fewer parameters**, while **6.18M-parameter Femto beats TimesFM-2.5, a model 37× its size**. Both comparisons hold for normalized MASE and CRPS.1, 4, 5

Nano, at **119 million parameters**, also beats Datadog’s Toto-2.0-2.5B on normalized MASE across both GIFT-Eval and TIME, despite Toto having **roughly 21× as many parameters**.3

The chart below compares accuracy with model size. On GIFT-Eval, each PDM model achieves lower error than the other displayed models with the same or fewer parameters. This is the **accuracy–size frontier** of this comparison.

The size comparison covers all 97 GIFT-Eval configurations using the October 31, 2026 snapshot of the official results.9 It includes models with documented parameter counts and is not an exhaustive leaderboard.

## Model size and benchmark comparisons

Nano achieves lower normalized MASE and CRPS than Timestral-209M on GIFT-Eval, and lower normalized MASE on TIME, with **43% fewer parameters**. Timestral-209M retains a small advantage on TIME CRPS and the two fev-bench metrics shown here. Pico, at 32.2M parameters, also achieves lower normalized GIFT-Eval MASE than Timestral-209M.

| Benchmark / metric | PDM-Nano · 119M | Timestral · 209M | 
|---|---|---|
| GIFT-Eval MASE ↓ | 0.637 | 0.658 | 
| GIFT-Eval CRPS ↓ | 0.433 | 0.446 | 
| TIME MASE ↓ | 0.625 | 0.626 | 
| TIME CRPS ↓ | 0.525 | 0.522 | 
| fev-bench SQL skill ↑ | 50.33% | 50.39% | 
| fev-bench MASE skill ↑ | 39.21% | 39.49% | 

## Evaluation settings and sources

The #1 results refer to normalized MASE among zero-shot foundation models as of November 1, 2026. Nano also leads GIFT-Eval’s zero-shot entries on normalized CRPS. Normalized errors and average per-configuration ranks are separate leaderboard measures.

We compare foundation models used without task-specific fitting or search. TIME sorts by normalized MASE by default, with Nano second overall behind TimeRSI-1.5, an adapted system using task-specific programs, historical backtests and forecast composition around frozen foundation-model weights.10

All PDM evaluations are zero-shot, with no benchmark test-data leakage. GIFT-Eval and TIME aggregate errors normalized by Seasonal Naive using a geometric mean; fev-bench uses its official skill-score aggregation. Lower MASE and CRPS are better; higher skill scores are better.

PDM uses up to 16,336 context steps, flip averaging and output constraints inferred from observed history. GIFT-Eval uses joint targets and available past covariates; TIME uses joint targets; fev-bench also uses past and future covariates. Our fev-bench submission includes results and reproduction code.8

Size ratios use total checkpoint parameter counts, rounded: Toto-2.0-2.5B has 2,454,281,792 parameters, about 20.6× Nano’s 119M; Timestral-209M has 209,312,256, about 6.5× Pico’s 32.2M; and TimesFM-2.5 has 231,289,280, about 37.4× Femto’s 6.18M.3, 4, 5 These are parameter ratios, not measured speedups.

Timestral’s comparison scores and settings come from its published model card.4 It reports unensembled results with up to 32,768 context steps. Parameter counts measure model size; these benchmarks do not establish relative inference speed or cost on matched hardware.

## Built in the UAE

Our work on prediction began in financial markets. Flashmind grew out of Infinite Field, where we spent years building predictive models for high-frequency trading. In trading, forecast errors affect decisions made with real money, and understanding uncertainty matters as much as estimating what comes next. We started Flashmind to make our work on prediction useful beyond markets.

The UAE has been our base for some years, and we’re building Flashmind from Dubai. We have also built out our own colocated infrastructure to train our models. We see great potential for the country as a hub for AI research and development. We want to contribute to that by developing models here, testing them openly and making them available to people building around the world.

## Try PDM on your data

Try an example, paste your data or draw a pattern below. No account is needed to get started. The full playground also lets you draw multiple series and add context such as prices or planned promotions. A free account includes **10,000 forecast calls per calendar month**, API keys and a usage dashboard, with the allowances explained in the limits table.

To use PDM in Python, install flashpdm. It returns forecasts with labeled quantiles and can convert them to pandas or NumPy. To use PDM through a compatible AI assistant, connect our MCP integration and supply your data. Developers can find the supported inputs and outputs in the API reference, or expand the code below to run the café comparison.

Nano, Pico and Femto are our first base models. We’ll share more about the next suite of PDM models soon.

## References

Leaderboard standings checked November 1, 2026.

- GIFT-Eval leaderboard. Normalized errors and per-configuration ranks. 1 2 3
- TIME leaderboard. Normalized MASE and CRPS comparisons.
- Datadog: Toto-2.0-2.5B. Model and checkpoint parameter count. 1 2
- Mistral AI: Timestral-209M. Model size, benchmark scores and evaluation settings. 1 2 3
- Google: TimesFM-2.5 checkpoint. Checkpoint used for the total parameter-count comparison. 1 2
- Flashmind GIFT-Eval submission. Results, reproduction notebook and maintainer verification. 1 2
- Flashmind TIME submission. Results, reproduction code and maintainer verification. 1 2
- Flashmind fev-bench submission. Results and reproduction code. 1 2
- GIFT-Eval comparison snapshot. Official results snapshot used for the accuracy–size chart, October 31, 2026.
- TimeRSI-1.5. Task-specific adaptation and forecast composition.
