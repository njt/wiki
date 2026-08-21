# Anomaly Detection

A beautifully minimal anomaly detection system: Welford's algorithm maintains a running mean and variance in constant memory, a key-value store tracks hourly event buckets, and anything more than 2 standard deviations from the mean triggers an alert. No configuration, no ML models, no training phase -- just math and a KV store.

---

## Key Quotes

> "If a new value is more than 2 standard deviations from the running mean, it's an anomaly."

## Key Themes

#observability #data-quality

The elegance here is in what's *not* included. No machine learning, no time-series database, no configuration wizard. Welford's algorithm needs exactly three numbers (count, mean, sum of squared deviations) to compute running variance with high numerical stability. Combined with hourly bucketing and a KV store with TTLs, the entire detection engine fits in a single file.

Three independent checks cover the practical bases: total volume anomalies (spike or crash), proportional anomalies (one event type dominating), and per-user anomalies (individual behavior shifts). The zero-backfill for quiet periods is a nice touch -- without it, the model would learn to ignore gaps, which is exactly when something is probably wrong.

The hardcoded z-score threshold of 2 (~5% false positive rate) is deliberately simple. It means roughly 1 in 20 normal hours will trigger a false alarm, which sounds bad until you realize most production anomaly detection systems produce far more noise from misconfigured thresholds.

## Critical Analysis

This is the right level of sophistication for most monitoring needs. The common mistake is reaching for ML-based anomaly detection when simple statistics would suffice -- and then spending more time tuning the ML system than you would have spent investigating false positives from a z-score check. The limitation is clear: it only detects anomalies in event counts, not in event content or latency distributions. [[Weird Machines in Transport Layer Security]] pushes the same instinct to a different substrate: its "sentinel" system composes TLS primitives to detect anomalous *handshake behavior* — anomalies in protocol state transitions rather than event counts — which is the content-level detection this z-score approach explicitly leaves out. But for the "is something weird happening right now?" question, this is hard to beat. The three-data-point minimum before alerting is a practical concession to cold-start noise.

---
*Sources: [[summary/anomaly-detection]]*
*Last updated: 2026-05-14*
