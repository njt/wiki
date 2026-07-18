---
url: https://michaelscodingspot.com/reduce-logging-costs/
title: "5 Strategies to Reduce Logging Costs"
author: Michael Shpilt
date_fetched: 2026-07-18
date_published: 2026-07-16
site: Michael's Coding Spot
tags: [observability, logging, cost-optimization, devops]
---

# 5 Strategies to Reduce Logging Costs

**Author:** Michael Shpilt (writing as "Michael's Coding Spot")

**Publication Date:** July 16, 2026

**Categories:** Observability | Tags: Logs, Observability, Obics

**Origin:** Originally posted on Obics.io

---

## Overview

The article argues that logging costs can consume "5 or 10 (or more) percent of your cloud host costs," which the author deems excessive. Most logs are "redundant and never queried," but teams still need them for incident response. The piece presents five strategies for balancing visibility with cost.

---

## Strategy 1: Sampling

Two approaches are described:

- **Head Sampling** — "you decide at the beginning what you want to sample and what not. e.g. sample every 10th trace."
- **Tail Sampling** — "you decide at the end," e.g., keeping only traces containing an error log. This requires buffering the entire trace or span before deciding.

The author notes that head sampling can be done via OpenTelemetry SDK config, while tail sampling requires an OTEL collector. A warning is given about vendor-side sampling features like DataDog exclusion filters, which "will randomly choose logs by the sampling percentage that you set, leaving traces with 'holes'." Additionally, DataDog's approach means you "keep paying for ingesting those logs, even if you don't pay for indexing them."

**Disadvantages:** Sampling discards information. "The events you care about most are often the rare ones" — a production bug occurring once in 10,000 requests, unusual user flows, or one-time failures. Counts become estimates, and signals become inconsistent across metrics, logs, and traces. The author notes that sampling "at best solves the problem partially," since you still pay to generate and transmit noisy data.

---

## Strategy 2: Tiered Storage

Vendors offer multiple storage tiers:

- **Default/standard tier:** Most expensive, fastest queries, supports alerts, dashboards, and metrics from logs.
- **Warm/middle tier:** Cheaper storage, slower queries, may lose monitor or metric-generation capabilities.
- **Cold/archive tier:** No query capability, supports "rehydration" — pulling logs back into the standard tier on demand, at a cost. Described as "a rare insurance."

A common pattern: keep logs in standard/warm tiers for short retention, then move to cold for longer compliance or incident-related retention. The article mentions "Telemetry Pipeline" solutions that act as a proxy between the application and the vendor, storing logs in cheap object storage. The trade-off is "adding another vendor, learning a new tool, and introducing more complexity."

**Disadvantages:** Always a trade-off between day-to-day analytics/incident response and cost. The author advocates a hybrid approach of short standard-tier retention followed by warm/cold migration but notes that "cross-tier analytics and even simple queries are a nuisance."

---

## Strategy 3: Clean Up and Optimize in Code

Logs accumulate over years across teams without cleanup, turning into "a dumpster fire of old and no longer relevant data." Techniques include:

- Removing duplications
- Aggregating repetitive logs into a single log or metric
- Unifying consecutive logs into a single entry
- Reducing log level to DEBUG or TRACE for logs not needed but not yet deletable
- Identifying and cutting overly verbose logs (e.g., inflated JSON serialization or long unnecessary explanations)

The author acknowledges that developers don't enjoy cleanup sprints — "Developers want to add features and work on the codebase, not to do house cleaning." A tool called [Obics](https://obics.io/early-access?utm_source=mcs&utm_medium=referral) is mentioned, which "uses agentic methods" to identify redundancies and optimization suggestions, even creating pull requests on demand. The author suggests a future where this process "might be entirely autonomous," though an engineer still needs to review.

---

## Strategy 4: Switch to a Cheaper Vendor

Moving to a lower-cost observability vendor addresses the expense problem but not the noise problem. The author notes many cheaper alternatives to "top tier" vendors like Splunk or Datadog exist. However, for "large companies and cross-functional teams, these migrations are notoriously expensive," especially without OpenTelemetry already in place. Even with full OTEL adoption, teams must migrate alerts, dashboards, and log-generated metrics, plus retrain everyone accustomed to the old tool.

---

## Strategy 5: Remove INFO Logs

Described as the worst-case scenario — when budgets are depleted, raise the minimum log level to WARN. "On the upside, you'll definitely save on your observability costs. On the downside, you're almost completely blind in production." The author calls this "a radical move for a radical situation."

---

## Conclusion

The article states there is no single silver bullet; different organizations need different combinations of strategies. Sampling sacrifices visibility, tiered storage complicates analysis, and switching vendors carries costly migration overhead.

The most sustainable approach, per the author, is to "reduce unnecessary telemetry before it ever leaves your application." Removing redundant logs saves CPU cycles, network bandwidth, storage, and future engineering time. Unlike sampling or tiering, "you're not throwing away potentially useful information. Instead, you stop generating data that nobody needs."

The closing argument is that observability cost optimization is shifting from seeking cheaper storage toward treating telemetry as code requiring continuous maintenance. Teams adopting that mindset will "spend less on observability" and end up with "cleaner, more useful data when incidents inevitably happen."
