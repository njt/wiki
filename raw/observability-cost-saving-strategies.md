---
url: https://michaelscodingspot.com/observability-cost-saving-strategies/
date_fetched: 2026-08-26
---

*Originally posted in Obics.io*

Observability vendors have been billing companies per data volume or per node count for many years. Both of those have exploded in the last decade and the vendors have been enjoying a rapid rise in income. But that also created frustration from the customers. The issue of cost is on everyone’s mind when it comes to observability and the #1 reason for switching vendors, even if you are perfectly happy with everything else.

While the observability platforms don’t want you to leave them, there’s no denying the conflict of interest. They are very happy with the rise in usage and with you paying them more money. So when you are asking for help from your vendor to save on cost, you are asking the fox to guard the henhouse

Here are 7 extremely effective strategies for cost reduction that you won’t likely be hearing from your vendor:

## Logs to Metrics Conversion

Many organizations pay a premium to index millions of logs (like HTTP requests or load balancer logs) only to use them like metrics. Visualizing dashboards or building alerts from aggregated values.

A much cheaper approach is a “Logs to Metrics” conversion. Instead of sending all raw logs to your vendor’s expensive indexed storage, intercept them at the OpenTelemetry Collector level or an edge proxy. Parse the log, increment a metric counter (e.g., `http_requests_total`) or record a latency histogram, and then filter out the source log. You’ll get the same dashboard visibility for a fraction of the cost.

Another (easier, but less effective) option is if your observability vendor supports converting logs to metrics. For vendors that bill for ingest + indexing, like DataDog, you’ll be saving the “indexing” part which is usually 80%-90% of the cost.

## Aggressively monitor metrics Cardinality

High cardinality is the silent killer of observability budgets. It appears unexpectedly and by its nature can blow up metrics cost.

Developers are often unaware of metrics billing or of sensitivity to cardinality. Attaching unbound values like `user_id`, `session_id`, or an ephemeral `pod_name` to a custom metric causes the number of unique timeseries to explode. So keep your eye on the ball at all times. The observability vendors aren’t too keen to alert about these spikes, so if you have a tool that does that (like Obics), it’s a big step in the right direction.

## Ask for BYOC

Bring Your Own Cloud (BYOC) is a deployment model where the telemetry (and often the query engine) is stored and runs inside your own cloud account. Your telemetry lands in your S3 / GCS / Azure buckets, and never leaves your perimeter. You pay your cloud provider for that storage and compute, and the vendor charges you a platform/license fee on top instead of a per-GB fee.

Cloud storage like S3 is much cheaper than an observability tools` storage rates. But this changes the *unit* you’re billed on. Instead of data volume it might be the number of nodes you’re instrumenting.

Both DataDog and Grafana recently introduced Bring Your Own Cloud (BYOC) offerings. I suspect that without a threat of leaving, you won’t be offered this solution on their initiative. But if you ask them explicitly, you can move to a much more beneficial plan.

For BYOC offerings that charge per node, if you are running a microservice architecture with lots of small nodes, that might not save that much. If you’re running big services or a monolithic architecture, it might pay out big time. In any case, a price simulation is free and worth a shot.

From the author I also build**Obics**Obics is a telemetry optimization tool to remove observability spend and noise. It finds redundancies in logs, metrics, and traces, and opens pull requests to fix them at the source. Take a look → 40–60% Lower observability costs Finance gets the savings. Without shutting down observability or losing visibility. 20–30% Faster MTTR DevOps and SRE get clean dashboards and alerts that fire on real issues. Zero Cleanup overhead Developers stay focused on product. No more being pulled into telemetry cleanup.

## Reduce metrics Data Points per Minute (DPM)

Many observability platforms calculate your metrics bill based on the frequency of the data points you send. DPM (Data Points per Minute), which essentially measures the resolution of your telemetry.

If you’re polling or flushing metrics every 10 seconds, you are generating 6 DPM per minute. For most applications, especially for long-term trending and high-level dashboards, 10-second resolution is an unnecessary overkill. By simply changing the flush interval in your agents, SDKs, or OpenTelemetry Collectors from 10 seconds to 60 seconds (1 DPM), you reduce your metric volume by over 80%.

One important caveat: this strategy only pays off if your vendor’s meter actually counts data points. Grafana Cloud does. Its billable series calculation multiplies your active series by your DPM. Same story for anyone charging per ingested sample or per GB, like most Prometheus-compatible backends.

Datadog, for example, is different. Custom metrics are billed per unique timeseries per hour, regardless of how many points you push into each one. Dropping from 10-second to 60-second flushes will slightly cut on network traffic and CPU, but it won’t reduce your Datadog metrics bill.

## Removing redundant logs at the source

The truth is that most logs are redundant and can be removed or optimized without sacrificing visibility. But that requires quite a lot of work and even some risk. You’ll need to identify the noisiest logs, understand the context, and figure out if they can be dropped or optimized. But that’s the only solution that solves the problem once and for all.

The real price of redundant logs is higher than your DataDog or Splunk bills. You’re also paying for excessive compute, higher network traffic, and even more LLM tokens (when you ask your agents to read the telemetry). Not to mention that noise causes higher MTTR in investigations, whether human or AI based.

In Obics, we’re making the process of finding and solving these redundancies basically free. Both in engineering effort and in price.

## Head & Tail Sampling

You don’t need to index 100% of your telemetry to know your system is healthy. Under load, a problem will keep repeating. Sampling allows you to keep representative data while discarding the rest.

- **Head-based sampling:**The decision to keep or drop telemetry data is made right at the source, at the start of a trace (in application instrumentation or with an agent/collector at the edge). For example, keep 10% of all traces. It can be a big cost saver, but you risk dropping rarely occurring problems.
- **Tail-based sampling:**The decision is deferred until the entire request or trace is completed. An aggregator (like the OTel Collector) buffers the data and evaluates it. If the trace was healthy all the way through and under 100ms, it keeps only 5% of them. If the request contained an error or was unusually slow, it keeps 100%. Tail-based sampling can be powerful, but also harder to implement. Not to mention that you can miss problems that don’t manifest as failed spans.

## Tiered log levels

Regular sampling has the problem that it misses load-based issues that affect an entire cluster or an entire datacenter. When that kind of issue happens, you often need to gather statistics from the full environment.

An effective pattern can be to retain a small subset of clusters or regions that are critical enough to justify full-fidelity INFO+ logs, while the rest of your environments can retain only WARN+ telemetry. In a way, you’re getting the best of both worlds.

The choice of the full-fidelity environment can either be by business importance or by condition. You can choose crowded regions (more compute per machine) or those with the worst internet connection to log everything with the idea that load-based problems will manifest in those clusters first.

So full visibility where it matters, cost-optimization everywhere else.

## Recap

To sum it up, vendors are businesses, and while they want you to be happy, they also want you to spend. Taking control of your observability costs requires a proactive approach on your part. That means dropping redundant telemetry at the source, optimizing resolution cardinality, converting expensive logs into efficient metrics, utilizing sampling, strategically tiering your log levels, or even moving to a BYOC model.

Depending on your use case, using one or several of the described techniques can drastically reduce your bill without sacrificing system visibility.
