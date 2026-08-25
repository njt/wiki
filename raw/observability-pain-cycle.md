---
url: https://nikola-petkovic.com/blog/2026/08/07/observability-pain-cycle/
date_fetched: 2026-08-25
---

If you used an observability product, you have probably experienced some version of this. The vendor asks you to ingest everything up front, and charge you the moment this huge volume of telemetry is ingested. Only then do you get a chance to explore that data and get some value from a small portion of it, when someone builds a dashboard, writes an alert, or goes exploring in the middle of an incident. So the spend is at full height from the first minute, regardless of when and to what extent you start to use it later. Fashions changed along the way: new protocols appeared (OTel), novel telemetry databases built on object storage and columnar layouts, AI happened. But the store-everything-up-front philosophy stayed firmly in place, protecting the vendor’s interest.

As a result, a large amount of telemetry noise sits around, eating your bill and slowing down your queries. Whether you are getting enough value for your spend is a second-order concern for the vendor. Their main worry is that the invoice doesn’t cross your pain point. And when it does, they hand you the tools to analyze and cut some telemetry. A couple of months later the cycle repeats: costs climb, and you are back to hunting for telemetry to trim. It is always the pain of a high bill that triggers the trimming exercise. This pattern is so ubiquitous that it deserves a name: **the Observability Pain Cycle**. It is the natural consequence of the store-everything-up-front philosophy that mainstream observability platforms were built on, and have done everything in their power to keep it that way.

## How store-everything-up-front became the default

The model was reasonable when it was introduced. On physical servers running monoliths, telemetry was low-volume and hand-instrumented. The challenge was to instrument and collect, not to store and query. *“Send us all your telemetry and query what you need”* was sound: storing some unused data cost a bit more, but it was acceptable.

A couple of things changed around a decade ago. Cloud, containers, microservices, and distributed tracing drastically increased the telemetry volume. And producing telemetry stopped being a concern. On the contrary, it became very easy to emit. Auto-instrumentation, built-in library metrics, exporters, and eBPF help you emit telemetry, often without writing instrumentation code at all. The store-everything-up-front model was never revisited. However, the cost became a problem impossible to ignore. So the vendors built some tools to “help” you when the bill starts to hurt.

The direct consequence of store-everything-up-front is a large amount of noise: telemetry you never asked for. It brings no real value, and it serves mainly that the vendor gets to charge you for it, telling you that “you never know whether you might need it eventually”. But the volume is not a joke anymore, an average vanilla Kubernetes cluster produces almost 100,000 time series out of the box.

## How it works

Let’s first take a look at what the system architecture looks like and how data flows through it. The architecture is sane, but the vendor-favoring defaults result in the customer pain points (shown in red).

This is how the story typically unfolds:

- **You onboard by following the vendor’s documented path.**Enable the integrations, turn on auto-instrumentation, scrape the whole cluster. The agent sees every metric, log line, and span at the source, which would be the right place to decide what to keep. But it ships with pass-through as the default. And if you flip it, there is a punishment: you can no longer put together dashboards and alerts, or browse through the telemetry that is available at the source.
- **Everything gets stored, only a small part gets used.**All telemetry lands in the backend, while dashboards and alerts (once they are created some time later) read a small, stable subset of it. Nobody tracks which stored series feed a panel or an alert, which have never been read, or which data would bring no practical value in possible future investigations.
- **The bill becomes a visible pain point that no longer stays under the radar.**An engineer is assigned to find out where the money went, and always finds the same things: metrics nobody reads, debug logs kept at full retention, cardinality bombs, namespaces onboarded and forgotten.
- **The trimming is coarse.**Drop these log levels, sample these traces at 10%, exclude these metric prefixes. In the absence of a clear link between stored signals and the value they bring, hunting for what to trim becomes a hard and error-prone exercise.
- **Nothing re-tunes the filters.**New services launch, auto-instrumentation lands on a hot path, traffic shifts. The filters stay tuned for a workload that no longer exists, and the volume silently grows back until the next invoice makes it visible. You feel the pain of the bill, and the Cycle starts over again.

## The shape of the cost

The shape of the cost over time follows directly from how the system works. The only signal that triggers a correction is the invoice, or a usage alert derived from the same meter. That signal is **late** (the volume is already ingested and paid for) and **coarse** (without a fine-grained correlation with the value it brings).

A system driven by such a signal settles on the only target it can express: spend just below the pain threshold. Trimming starts when the bill hurts and stops when it doesn’t, rarely when most of the waste is gone. And even if you do all the due diligence and cut the majority of the waste, it is still a one-time effort. A couple of months down the road, it no longer corresponds to the new reality.

## Tactics to avoid the change

**“You never know which signal the next incident will need.”** This is the mother of all arguments, pulled by every observability sales person. It is meant to frighten you if you dare to ingest less telemetry. And it silently ignores your expertise, and the fact that it is you who understands your context the best. The argument is true, but only for a thin slice of your telemetry. The vast majority of automatically emitted telemetry will never be used. It works because the platform gives you no way to tell which is which, so “keep everything” is the only decision that requires no analysis.

**“Storage is cheap” is missing the point.** Bytes at rest are only a fraction of cost. You also pay for ingestion and indexing compute, for queries running over an ever larger dataset, and for network egress. And slower queries and harder analysis are a price you pay every time you read the data because the bigger haystack does not make it easier to find the needle.

**AI features sit downstream of the problem.** Anomaly detection, incident correlation, natural-language querying: all useful, but they run on top of data that is already collected, and have no say in whether it should have been collected at all. The noise hurts them too: every irrelevant piece of telemetry makes the search space bigger and increases the chance of AI hallucination.

**The classic cost controls decide  how much data to keep, not which data matters.** Sampling, shaping pipelines, cardinality pruning, usage caps: they all trim volume without knowing the value. Some exceptions exist. For example, tail-based trace sampling (keep the slow and failed traces), if properly implemented (which it often isn’t), is a real per-signal value judgment. It is possible because traces carry a value marker the pipeline can read: latency and error status. Metrics and logs have no such marker. Their value lies in whether anything downstream uses them, and that is exactly the information these pipelines never look at.

**“Transparent billing” doesn’t help either.** Every line item is accurate, but none of them answers the real question: *which of these signals is worth paying for?* Without that information, every trimming decision is a trade-off between cutting too much (dropping the series the next incident needs) and cutting too little (keeping noise that keeps costing you). Teams reasonably choose to cut too little because a missing telemetry during an incident is visible and somebody gets blamed for it.

## Conclusion

**The Observability Pain Cycle** is not an accident, it is the natural outcome of the store-everything-up-front model. The model also largely ignores you, the user. You know your systems, you know the context, and you should be the one deciding which questions you want your observability to answer. Instead, the vendor’s defaults decide what gets collected, and the bill decides what gets cut. The one party who actually knows what is valuable has a little say in the loop.

And the problem is getting worse, not better. In the AI era, telemetry is exploding by another order of magnitude: AI workloads and agents emit their own telemetry on top of everything else, and produce it faster than any human team ever did. The pain that was already hard to keep under control is now outgrowing the tools built to manage it. This will not fix itself, and I doubt it will be fixed by those who profit from the noise.

← All posts
