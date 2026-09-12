---
url: https://analytics.fixelsmith.com/posts/sql-fraud-patterns/
title: Six SQL patterns I use to catch transaction fraud
author: Fixel Smith
date_fetched: 2026-05-22
date_published: 2026-05-14
topics:
  - databases-and-data
---

# Six SQL patterns I use to catch transaction fraud

**Author:** Fixel Smith — "experienced Program Integrity Analyst working in public-sector data"

**Published:** May 14, 2026 (originally at analytics.fixelsmith.com, also on dev.to)

**Disclaimer:** The author states they work on a program-integrity team, using "generic transaction tables and made-up scenarios" unrelated to their actual work.

## Opening Context

"Fraud detection in transaction data is mostly SQL" — not ML, graph DBs, or analyst hype. The author works with government benefit programs but notes the patterns transfer to credit cards, healthcare claims, e-commerce, and POS systems. The six patterns are presented roughly in the order they'd build them on a new dataset.

## Pattern 1: Velocity

Someone with a stolen card transacts rapidly to drain funds before detection.

```sql
SELECT
  cardholder_id,
  date_trunc('hour', timestamp) AS hour_bucket,
  count(*) AS tx_count,
  min(timestamp) AS first_tx,
  max(timestamp) AS last_tx
FROM transactions
WHERE timestamp >= current_date - INTERVAL '30 days'
GROUP BY 1, 2
HAVING count(*) > 10;
```

Tuning involves adjusting window size and count threshold. The author runs "a 1-minute, 5-minute, and 1-hour version in parallel" because different fraud patterns surface at different time scales — card-testing rings hit in seconds while benefits trafficking might span an afternoon.

False positives include vending machine route operators and people reloading prepaid cards in bulk. The recommendation is maintaining a whitelist after initial passes.

Sliding-window approach:

```sql
SELECT
  cardholder_id,
  timestamp,
  count(*) OVER (
    PARTITION BY cardholder_id
    ORDER BY timestamp
    RANGE BETWEEN INTERVAL '5 minutes' PRECEDING AND CURRENT ROW
  ) AS tx_in_last_5min
FROM transactions
QUALIFY tx_in_last_5min >= 5
ORDER BY cardholder_id, timestamp;
```

`QUALIFY` works in Snowflake, BigQuery, Databricks, Teradata; Postgres requires wrapping in a CTE with an outer filter.

## Pattern 2: Impossible Travel

A single card appearing in distant locations within minutes indicates cloning — "the most uncontroversial fraud signal you'll find."

```sql
WITH ordered_tx AS (
  SELECT
    cardholder_id,
    timestamp,
    location,
    LAG(timestamp) OVER (PARTITION BY cardholder_id ORDER BY timestamp) AS prev_ts,
    LAG(location)  OVER (PARTITION BY cardholder_id ORDER BY timestamp) AS prev_loc
  FROM transactions
)
SELECT
  cardholder_id,
  prev_ts  AS first_tx,
  timestamp AS second_tx,
  prev_loc  AS first_location,
  location  AS second_location,
  EXTRACT(EPOCH FROM (timestamp - prev_ts)) / 60 AS minutes_apart,
  haversine(prev_loc, location)                  AS miles_apart
FROM ordered_tx
WHERE prev_ts IS NOT NULL
  AND prev_loc <> location
  AND haversine(prev_loc, location)
        / nullif(EXTRACT(EPOCH FROM (timestamp - prev_ts)), 0)
        * 3600 > 600;
```

The 600 mph threshold approximates "faster than a plane could possibly do it" (commercial jet cruise ~575 mph). Tightening to 100 mph catches suspicious ground travel but flags legitimate airline passengers and family road trips.

Variations: "Two distant cities, same state, inside 5 minutes" for local cloning rings; "Multiple ZIP codes inside an hour" for skimmer rings; "Border crossings inside 10 minutes" for international rings.

## Pattern 3: Amount Anomalies

Certain dollar amounts appear disproportionately in fraud scenarios and rarely in legitimate transactions.

```sql
SELECT cardholder_id, timestamp, amount, merchant_id
FROM transactions
WHERE
  (amount >= 99.50  AND amount < 100.00)
  OR (amount >= 499.50 AND amount < 500.00)
  OR amount IN (1.00, 5.00, 10.00)
ORDER BY cardholder_id, timestamp;
```

Round dollar amounts like $1.00, $5.00, and $10.00 are "almost always card tests" — someone verifying a stolen number before reselling it. "Real cardholders almost never buy something for exactly $1.00" — coffee costs $4.73, gas costs $52.81. The roundness itself is the signal.

Amounts just below thresholds tell a different story: $99.99 often sits where $100 triggers ID checks; $499.99 sits below $500 daily ATM caps. The fraudster "knows the rules and is staying under them."

For benefits transactions, round-number patterns aren't as useful — the signal there is duplicate recipients instead.

## Pattern 4: Suspicious Merchants

A compromised card reader (skimmer) produces dozens of fraud cases — many unrelated cards spending unusual amounts at the same merchant in a short window.

Static threshold version:

```sql
SELECT
  merchant_id,
  date_trunc('hour', timestamp) AS hour_bucket,
  count(DISTINCT cardholder_id) AS unique_cards,
  count(*) AS total_tx,
  sum(amount) AS total_amount
FROM transactions
WHERE timestamp >= current_date - INTERVAL '7 days'
GROUP BY 1, 2
HAVING count(DISTINCT cardholder_id) > 20
  AND sum(amount) > 5000
ORDER BY total_amount DESC;
```

The problem: "A Costco does that in 90 seconds. A used bookshop, never." So the better approach compares each merchant against its own historical baseline:

```sql
WITH merchant_hourly AS (
  SELECT
    merchant_id,
    date_trunc('hour', timestamp) AS hour_bucket,
    count(DISTINCT cardholder_id) AS unique_cards
  FROM transactions
  WHERE timestamp >= current_date - INTERVAL '60 days'
  GROUP BY 1, 2
),
with_baseline AS (
  SELECT
    *,
    avg(unique_cards) OVER (
      PARTITION BY merchant_id
      ORDER BY hour_bucket
      ROWS BETWEEN 168 PRECEDING AND 1 PRECEDING
    ) AS rolling_avg_cards
  FROM merchant_hourly
)
SELECT *,
  unique_cards / nullif(rolling_avg_cards, 0) AS spike_ratio
FROM with_baseline
WHERE unique_cards > rolling_avg_cards * 3
ORDER BY spike_ratio DESC;
```

The 168 represents seven days of hourly buckets — chosen because "daily and weekly seasonality matters" (Tuesday 2pm at a coffee shop differs from Saturday 9am). Three times the historical average is the starting threshold, "loose enough not to drown you in alerts but tight enough to flag the actually weird hours."

## Pattern 5: Off-Hours

Most people spend money during consistent hours. A 9-to-5 worker's card used at 3am for gas is likely stolen.

```sql
WITH cardholder_hour_pattern AS (
  SELECT
    cardholder_id,
    EXTRACT(HOUR FROM timestamp) AS hour_of_day,
    count(*) AS tx_count
  FROM transactions
  WHERE timestamp >= current_date - INTERVAL '90 days'
  GROUP BY 1, 2
),
cardholder_normal AS (
  SELECT
    cardholder_id,
    min(hour_of_day) FILTER (WHERE tx_count >= 2) AS earliest_hour,
    max(hour_of_day) FILTER (WHERE tx_count >= 2) AS latest_hour
  FROM cardholder_hour_pattern
  GROUP BY 1
)
SELECT t.cardholder_id, t.timestamp, t.amount, t.merchant_id
FROM transactions t
JOIN cardholder_normal cn USING (cardholder_id)
WHERE EXTRACT(HOUR FROM t.timestamp) NOT BETWEEN cn.earliest_hour AND cn.latest_hour
ORDER BY t.timestamp DESC;
```

The "two or more in that hour" filter prevents a single late-night purchase from becoming part of the cardholder's "normal" range. Requiring two purchases in a given hour across 90 days "sets the bar at 'actually a habit' instead of 'happened once.'"

Drawback: New accounts lack history, so the author falls back to "global hour patterns or just skip this pattern entirely" until sufficient data accumulates.

## Pattern 6: Window Functions for Chained Signals

This isn't a standalone pattern but an infrastructure layer making the other five patterns composable.

```sql
SELECT
  cardholder_id,
  timestamp,
  amount,
  merchant_id,

  timestamp - LAG(timestamp) OVER w AS time_since_last,

  CASE WHEN merchant_id <> LAG(merchant_id) OVER w
       THEN 'changed' ELSE 'same' END AS merchant_change,

  sum(amount) OVER (
    PARTITION BY cardholder_id
    ORDER BY timestamp
    RANGE BETWEEN INTERVAL '24 hours' PRECEDING AND CURRENT ROW
  ) AS running_24h_total,

  ROW_NUMBER() OVER (
    PARTITION BY cardholder_id, date(timestamp)
    ORDER BY timestamp
  ) AS tx_of_day

FROM transactions
WINDOW w AS (PARTITION BY cardholder_id ORDER BY timestamp)
ORDER BY cardholder_id, timestamp;
```

Once these columns are materialized, fraud rules collapse to filter expressions. Example — hunting card-testing rings (small charges, many merchants, minutes apart):

```sql
SELECT *
FROM tx_with_windows
WHERE tx_of_day >= 5
  AND time_since_last < INTERVAL '60 seconds'
  AND merchant_change = 'changed';
```

"Three filters. That's it." The author emphasizes this shrinks the iteration loop from weeks to hours because analysts can express fraud hypotheses "as SQL filters instead of engineering tickets."

## Putting It Together

Each pattern alone is insufficient — velocity has false positives, geographic impossibility misses same-metro fraud, amount anomalies don't apply outside card-testing, off-hours needs history. The approach is running all patterns and scoring each transaction across signals — failing 3–4 means almost certainly fraud; failing one "might be your grandma being weird with her debit card on vacation."

Advice for newcomers: "start with pattern 1." For teams already running patterns 1–5, invest in pattern 6 — the window-function primitives — because "every analyst on your team will use them once they exist."

## Things Left Out

- **NULL handling:** Legacy systems use sentinel values like `9999-12-31` for "no end date" or `0001-01-01` for "no start date" — filtering with `IS NULL` silently misses these rows.
- **False positives:** "Auto-blocking on a single rule is how you lose customers." Human review with a feedback loop is essential.
- **Privacy compliance** when PII is involved.
- **Cost:** "Window functions with big partitions are not cheap." Filter date ranges first, then apply windows.

## Upcoming Topics

Eight window-function tricks beyond LAG and ROW_NUMBER; detecting fraud rings as a social-graph problem; what belongs on a fraud team's dashboard; how to fix noisy fraud alerts without merely raising thresholds.
