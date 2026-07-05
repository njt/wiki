---
url: https://uriv.me/blog/anomaly-detection-with-welford-and-kv
date_fetched: 2026-07-05
backfilled: true
---

# anomaly detection with nothing but math and a key-value store

I run a lot of projects. Things break in all of them, and I wanted a single service that would email me when something weird happens, without having to configure what "weird" means for each one.

The result is anomalisa, an open source event anomaly detector. You send events, it learns what normal looks like, and emails you when something deviates. No thresholds, no dashboards, no time-series database. The entire storage layer is Deno KV.

This post is about how the detection works.

## the core problem

You have a stream of events. "signup", "purchase", "error", whatever. They come in at some rate that varies by hour. You want to know when that rate is significantly different from what's normal, but you don't want to store every data point, and you don't want to configure anything upfront.

The answer is an online algorithm that maintains a running statistical model and updates it incrementally.

## welford's algorithm

The textbook way to compute variance is to collect all values, compute the mean, then compute the sum of squared deviations. That requires storing every value.

Welford's algorithm does it in a single pass with constant memory. You maintain
three numbers: `n` (count), `mean`, and `M2` (sum of squared deviations from the
running mean). For each new value:

```
const updateStats = (stats: Stats, value: number): Stats => {
  const n = stats.n + 1;
  const delta = value - stats.mean;
  const mean = stats.mean + delta / n;
  const delta2 = value - mean;
  const m2 = stats.m2 + delta * delta2;
  return { mean, m2, n, lastBucket: stats.lastBucket };
};
```
Standard deviation is then `sqrt(M2 / (n - 1))`. That's Bessel's correction for
sample variance, which matters when you have few data points.

The key insight is that `delta` is computed *before* updating the mean, and
`delta2` is computed *after*. This is what makes the numerics stable. A naive
running variance that tracks sum and sum-of-squares separately accumulates
floating point error.

With these three numbers you can compute z-scores on the fly. If a new value is more than 2 standard deviations from the running mean, it's an anomaly.

## hourly buckets

Events get counted in hourly buckets. Each hour, the total count for the
previous hour gets fed into the Welford model. The bucket key is just the ISO
timestamp truncated to the hour: `2026-04-05T14`.

This gives you a natural unit of measurement. "Your signup event usually gets ~50 per hour, this hour it got 3." The alternative is sliding windows, which are more responsive but harder to reason about and more expensive to maintain.

When a new hour starts and no events have come in, that's important information. Zero is a valid count that should update the model. But what if there are gaps where no events arrive for multiple hours? The system backfills zeros:

```
const updateStatsWithZeros = (stats: Stats, count: number): Stats =>
  Array.from({ length: count }).reduce<Stats>(
    (s) => updateStats(s, 0),
    stats,
  );
```
If the last bucket was 3 hours ago, two zeros get fed in before the current count. This prevents the model from ignoring quiet periods and keeps the mean honest.

## three detection modes from one stream

Every event that comes in contributes to three independent checks:

**Total count z-score.** The simplest one. Count events per hour, compare to the
running model. Works in both directions, it catches drops as well as spikes.
Your checkout event going from 50/hour to 3/hour is just as interesting as it
going to 200.

**Percentage spike.** If one event type suddenly represents a much larger
fraction of total events than usual, that's suspicious even if the absolute
counts look fine. This catches the case where errors go from 2% to 30% of your
traffic while total volume stays flat.

**Per-user anomalies.** Track the maximum single-user event count per hour and
build a separate Welford model for that. One user generating 100x their normal
volume is worth knowing about, whether it's a bug, a bot, or abuse.

Each mode has its own stats stored under a different KV key prefix. The total count and percentage checks run on bucket transitions (when a new hour starts). The per-user check runs on every event, since you want to catch a user spike before the hour ends.

## why KV is enough

The whole storage model is just KV keys with TTLs:

- `["counts", projectId, eventName, bucket]`→ hourly count (7-day TTL)
- `["stats", "total", projectId, eventName]`→ Welford state for total counts
- `["stats", "perUser", projectId, eventName]`→ Welford state for per-user max
- `["userCounts", projectId, eventName, bucket, userId]`→ per-user hourly count (7-day TTL)
- `["anomalies", projectId, eventName, bucket, metric, userId]`→ detected anomaly (30-day TTL)

No relational queries. No joins. No migrations. The Welford state is 4 numbers. Everything else is counters and flags. TTLs handle cleanup automatically.

The anomaly key includes the bucket and metric type, so the same anomaly can't be recorded twice. The store operation uses an atomic check-and-set:

```
const storeAnomaly = async (anomaly: Anomaly): Promise<boolean> => {
  const existing = await (await getKv()).get(anomalyKey(anomaly));
  return existing.value ? false : (await (await getKv()).atomic()
    .check(existing)
    .set(anomalyKey(anomaly), anomaly, { expireIn: anomalyTtlMs })
    .commit()).ok;
};
```
If two concurrent requests both detect the same anomaly, only one write succeeds. This is important because the system sends email alerts, and you don't want duplicate emails for the same spike.

## the tradeoffs

The z-score threshold of 2 is hardcoded. That's roughly a 5% false positive rate for normally distributed data. Event counts aren't perfectly normal, but it's close enough in practice. I tried making it configurable and realized that nobody, including me, wants to pick a threshold. The whole point is zero config.

The system needs at least 3 data points before it starts alerting. That means the first 3 hours of a new event type are a learning period. This is intentional. Alerting on "your second data point is different from your first" would be useless.

It won't catch everything. If your system fails in a way that doesn't affect
event counts, you're on your own. But most real failures *do* show up as
something spiking or dropping, and the simplicity of the approach means there's
almost nothing to debug when it doesn't work as expected.

The entire detection engine is one file. That's the best argument for this design. When your anomaly detection system itself breaks, you want to be able to read the whole thing in five minutes.

## try it

anomalisa is free and open source. Three lines to integrate:

```
import { sendEvent } from "@uri/anomalisa";
await sendEvent({
  token: "your-token",
  userId: "user-123",
  eventName: "purchase",
});
```
GitHub: github.com/uriva/anomalisa
