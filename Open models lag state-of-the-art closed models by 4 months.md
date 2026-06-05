---
url: "https://epoch.ai/data-insights/open-closed-eci-gap"
title: "Open models lag state-of-the-art closed models by 4 months"
author: "Jack Edwards and Luke Emberson"
date_fetched: 2026-06-05
date_published: 2026-05-29
source: "Epoch AI"
tags:
  - "#benchmark"
  - "#concept"
  - "#AI-research"
---

# Open models lag state-of-the-art closed models by 4 months

Epoch AI quantifies the open-closed capability gap using their Capabilities Index (ECI): from January through May 2026, the best open-weight models trailed frontier closed models by ~4 months on average — an 8-point ECI gap, roughly the difference between GPT-5 and GPT-5.5. The gap has widened slightly from the ~3 months observed in the prior two years (Jan 2023–Oct 2025). Both limitations the authors flag — public benchmark overfitting by open models and selection bias from unreleased frontier models — point in the same direction: the true gap is almost certainly *larger*.

## Key Quotes

> "The average time lag was around four months. The average ECI gap was eight points — similar to the difference between GPT-5 and GPT-5.5."

Four months doesn't sound like much, but "eight ECI points" reframes it in capability-space rather than calendar time. That's a full generation gap.

> "Open-weight models tend to perform worse on private benchmarks compared to closed models."

This is the most important sentence in the piece. It means public benchmark scores for open models are inflated relative to their real-world capability — they're overfitting the test, not just behind on the timeline. The 4-month estimate is a *floor*, not a point estimate.

> "If the bar were raised so the open-weight model's point estimate must strictly exceed the closed model's... the average gap would grow to six months."

The choice of a 5% bootstrap overlap threshold rather than a strict-exceed criterion is methodologically defensible but policy-consequential. Changing one significance threshold adds two months to the gap. The headline number is sensitive to choices the authors made conservatively.

## Key Themes

- **#benchmark** — ECI as an aggregate capability metric; the bootstrap methodology for handling uncertainty
- **#concept** — The open-closed gap as a structural feature of the AI landscape, not a transient one
- **#pattern** — Public benchmark overfitting as a systematic bias that makes open models look closer than they are
- **#pattern** — Selection bias from unreleased models: what you can't measure may be the most important part

## Critical Analysis

**The honest limitation is the most important finding.** When both of your identified biases push in the same direction — both understating the gap — the headline number becomes a lower bound, not an estimate. The true 4-month figure is better understood as "at least 4 months, probably more." Epoch AI is admirably transparent about this, but the framing still centers the number rather than the direction of error.

**The bootstrap threshold choice matters more than the authors let on.** Moving from the 5% overlap criterion to strict exceedance bumps the gap from 4 to 6 months — a 50% increase from changing one methodological knob. This isn't a criticism of their choice (the 5% threshold is reasonable), but it reveals how much of the finding lives in the methodology, not the data.

**The ECI itself is doing a lot of work here.** Aggregate capability indices compress multidimensional model performance into a single number. This is useful for trend-spotting but hides *which* capabilities are lagging. Are open models behind on reasoning? Coding? Multilingual? The ECI doesn't say, and that matters for anyone deciding whether the gap is practically significant for their use case.

**The widening trend deserves more attention.** Going from 3 months to 4 months may seem like noise, but if it continues, it suggests a structural dynamic: closed labs are pulling ahead, not just staying ahead. The compute thesis ([[Zheng Dong Wang's 2025 Letter]]) would predict exactly this — those with more capital build more capable models faster.

**For the "just use open models" crowd:** This data is a useful calibration. Open models *are* catching up, but they're running on a treadmill where the frontier keeps moving. If your use case requires SOTA capability, you're still on the closed-model treadmill whether you like it or not.

## See Also

- [[2025 in LLMs]] — Simon Willison's landscape survey, broader model ecosystem context
- [[Benchmark Exploitation]] — The gaming of public benchmarks that Epoch flags as a limitation
- [[Recent Developments in LLM Architectures]] — The architectural innovations that might close (or widen) the gap
- [[Granite 4.1]] — IBM's open model, Apache 2.0, an example of open-weight push
- [[Muse Spark]] — Meta's frontier closed reasoning model — the kind of thing widening the gap
- [[Step 3.7 Flash]] — Open model achieving 97% of Opus 4.6 coding performance at 1/9th cost
- [[Notes from the AI Now Summit by Mistral]] — The "model alone isn't enough" thesis and the open/closed strategic landscape
- [[Zheng Dong Wang's 2025 Letter]] — The compute thesis predicting exactly this gap dynamic
- [[Self-Hosted LLMs]] — What it takes to run these models yourself
- [[Demystifying Evals for AI Agents]] — The eval problem that underpins all capability measurement

---
*Source: Epoch AI Data Insights, May 29, 2026. Fetched June 5, 2026.*
