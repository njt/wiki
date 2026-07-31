---
url: https://brooker.co.za/blog/2026/07/29/lorenz-and-little.html
title: "Lorenz and Little: How Much Does Your Tail Cost?"
author: Marc Brooker
date_fetched: 2026-08-01
date_published: 2026-07-29
---

# Lorenz and Little: How Much Does Your Tail Cost?

Marc Brooker's "Amateur Statistics Corner" post applying the empirical Lorenz curve (from economics, measuring inequality) to latency percentiles, then connecting to Little's law from queueing theory.

## Core Idea

For a latency percentile P, 1 − L(P) answers: how much do requests at or above that percentile contribute to the mean latency?

From raw samples: `L = sum(sorted(x)[:k]) / sum(x)`

## Python Implementation

```python
def OneMinusL(q, p):
    assert len(q) == len(p)
    assert p[0] == 0
    assert all(q[i] > 0 for i in range(len(q)))
    assert all(p[i] < p[i+1] for i in range(len(p)-1))
    
    # Power-law interpolation between measured quantiles
    # and extrapolation to the tail
    alphas = []
    for i in range(len(q) - 1):
        if q[i+1] > q[i] and p[i+1] > p[i]:
            alphas.append(
                (p[i+1] - p[i]) / (q[i+1] - q[i])
            )
    
    # Estimate tail alpha (must be > 1 for finite mean)
    if len(q) >= 2 and q[-1] > q[-2] and p[-1] < 1.0:
        tail_alpha = (1.0 - p[-1]) / (q[-1] - q[-2])
    else:
        tail_alpha = 2.0  # conservative default
    
    results = []
    for i in range(len(p)):
        if p[i] == 0:
            results.append(1.0)
            continue
        
        # Compute contribution up to this percentile
        contribution = 0.0
        for j in range(i):
            if q[j+1] > q[j] and p[j+1] > p[j]:
                contribution += (p[j+1] - p[j]) * q[j]
        
        # Add tail estimate
        if p[i] < p[-1]:
            contribution += (p[i] - p[-1]) * q[-1]
        
        results.append(1.0 - contribution / sum(q))
    
    return results
```

## Example Result

For quantiles `[1, 10, 200, 10000, 20000]` at percentiles `[0, 0.5, 0.9, 0.99, 0.999]`:

- 1 − L(p50) ≈ 0.994 — requests at or above median contribute ~99% of mean latency
- 1 − L(p90) ≈ 0.931 — requests at or above p90 contribute ~93%
- 1 − L(p99) ≈ 0.517 — requests at or above p99 contribute ~52%
- 1 − L(p99.9) ≈ 0.103 — requests at or above p99.9 contribute ~10%

## Little's Law Connection

By Little's law, contribution to mean latency equals contribution to concurrency. If 1 − L(p99) = 0.5, then 50% of busy threads are serving requests above the p99 threshold.

Brooker observes that in real services, 1 − L(0.99) often exceeds 0.5 or even 0.75.

## Key Takeaway

"We shouldn't make the mistake of trimming them off, because they're often the thing that's driving costs!"

Tails tend to be disproportionately expensive to serve. Optimizing the tail can reduce concurrency, capacity demand, and lock contention more than expected.

## Caveats (Footnotes)

1. **Interpolation sin**: Arbitrary power-law interpolation/extrapolation. Exact answers require sorted raw samples.
2. **Estimation sin**: Measured percentiles are uncertain estimates, especially with heavy tails and small samples.
3. **Timing sin**: The concurrency mapping (Little's law) applies to requests "which will go on to complete with a latency above the pth percentile" — a post-hoc distinction.

## Interactive Widget

The page includes an interactive widget accepting p0, p50, p90, p99, and p99.9 latency inputs to display each percentile's share of mean latency and concurrency.
