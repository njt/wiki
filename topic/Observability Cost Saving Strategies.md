# Observability Cost Saving Strategies

Michael Shpilt's seven-strategy field manual for cutting observability bills without losing visibility. The piece opens with the structural conflict — vendors bill per volume or per node and profit from your usage, so "asking the fox to guard the henhouse" — then walks a ladder from cheap interception tricks (logs-to-metrics, DPM reduction) through architectural moves (BYOC) to the only fix that ends the problem (removing redundant logs at the source). Its most distinctive contribution is meter-awareness: several strategies only pay off if they match how your specific vendor actually bills.

---

## Key Quotes

> The issue of cost is on everyone's mind when it comes to observability and the #1 reason for switching vendors, even if you are perfectly happy with everything else.

The premise. Cost, not capability, is the churn driver — which is exactly why vendors have no incentive to hand you cost-saving advice unprompted.

> So when you are asking for help from your vendor to save on cost, you are asking the fox to guard the henhouse.

The thesis in one line, and the article's real subject is the tension hiding inside it: several of his own strategies (BYOC, and the vendor-side logs-to-metrics conversion) require asking the vendor anyway. The fox may guard the henhouse, but it's the only one with the keys.

> High cardinality is the silent killer of observability budgets. It appears unexpectedly and by its nature can blow up metrics cost.

Cardinality named as an emergent failure mode: it's not the metric you intended that costs you, it's the unbound dimension (`user_id`, `session_id`, `pod_name`) a developer attached without knowing it's a billing input. Developers are "often unaware of metrics billing or of sensitivity to cardinality."

> Cloud storage like S3 is much cheaper than an observability tools' storage rates. But this changes the *unit* you're billed on. Instead of data volume it might be the number of nodes you're instrumenting.

The sharpest architectural insight. BYOC doesn't just lower the rate, it moves the meter — and per-node pricing punishes microservices ("lots of small nodes") while rewarding monoliths and big services. He's unusually honest that it "might not save that much" depending on topology.

> By simply changing the flush interval ... from 10 seconds to 60 seconds (1 DPM), you reduce your metric volume by over 80%.

The 80% number is the headline, but the footnote is the value: "this strategy only pays off if your vendor's meter actually counts data points. Grafana Cloud does." And then the killer caveat:

> Datadog, for example, is different. Custom metrics are billed per unique timeseries per hour, regardless of how many points you push into each one. Dropping from 10-second to 60-second flushes ... won't reduce your Datadog metrics bill.

This is the article's rarest quality — cost advice keyed to the actual billing model. Most "reduce your observability bill" posts treat all vendors as interchangeable; Shpilt tells you which lever works for which meter, and admits when the lever does nothing.

> The real price of redundant logs is higher than your DataDog or Splunk bills. You're also paying for excessive compute, higher network traffic, and even more LLM tokens (when you ask your agents to read the telemetry).

The 2026-era extension of "telemetry as code": redundant logs tax not just storage but CPU, egress, and — newly — the context windows of AI agents that read telemetry. Noise raises MTTR "whether human or AI based."

> So full visibility where it matters, cost-optimization everywhere else.

The tiered-log-levels summary, and the article's quietest insight: the right answer to "you don't know which signal you'll need" is not to keep everything everywhere, but to keep full fidelity only where load-based problems will manifest first.

---

## Key Themes

- **#concept Meter-aware cost optimization** — the through-line. DPM reduction works on Grafana Cloud, not Datadog; BYOC changes the unit, not just the rate; logs-to-metrics saves the *indexing* portion on ingest+index vendors. Cost tactics are vendor-meter-specific, and advice that ignores the meter is noise.
- **#pattern Logs-to-metrics conversion** — intercept raw logs at the OTel Collector or edge proxy, increment a counter or record a histogram, drop the source log. The cheapest win for teams indexing logs they only ever aggregate.
- **#pattern BYOC (Bring Your Own Cloud)** — telemetry stored and queried in your own cloud account, vendor charges platform/license instead of per-GB. The architectural option that renegotiates the pricing model rather than trimming within it.
- **#concept Cardinality as cost driver** — unbound label values explode unique timeseries; vendors don't proactively alert on it. Monitoring cardinality is a discipline, not a feature.
- **#pattern Head vs. tail sampling** — head-based is cheap but risks dropping rare events; tail-based defers the decision until the trace completes and keeps 100% of errors and slow traces, 5% of healthy ones.
- **#tool OpenTelemetry Collector** — the recurring interception point where logs become metrics and flush intervals get tuned.
- **#person Michael Shpilt** — author of Michael's Coding Spot and builder of Obics, writing from the customer side of the vendor relationship.

---

## Critical Analysis

**This is a companion to, not a replacement for, [[Reduce Logging Costs]].** Two of the seven strategies (removing redundant logs, head/tail sampling) are repeats from that earlier five-strategy piece, and the "reduce before it leaves the application" thesis is carried over verbatim. The genuinely new material is the meter-aware layer: BYOC, DPM reduction with the Datadog caveat, cardinality monitoring, logs-to-metrics, and tiered log levels. Read together, the two articles form one coherent program — reduce at the source, tune resolution, convert expensive signals into cheap ones, sample what you can, and renegotiate the pricing model where you can.

**The fox/henhouse framing is right and also self-serving.** Shpilt is transparent that he sells Obics, a tool that "finds redundancies in logs, metrics, and traces, and opens pull requests to fix them." That doesn't invalidate the advice — the meter-aware details stand on their own — but it does explain the article's asymmetry: the strategies his product automates (redundant-log removal) get the "only solution that solves the problem once and for all" treatment, while the strategies it doesn't (BYOC, which you must negotiate with a vendor) are framed as hard-to-get.

**The Datadog caveat is the most valuable paragraph in the piece, and it indicts the rest of the genre.** Most cost-cutting advice implies a single lever works everywhere; Shpilt's own article shows why that's false. This is the lens [[The Observability Pain Cycle]] should be read through: the cycle persists partly because the tactics people reach for are generic, and a generic tactic applied to the wrong meter saves nothing.

**The "LLM tokens" line is a quiet 2026 signpost.** Redundant logs now cost more than storage and compute — they bloat the context of AI agents that read telemetry, and raise MTTR whether the investigator is human or model. That connects the observability-cost problem to the same agent-economics thread running through [[The Three Pillars of Observability]]'s question about agents as first-class consumers of telemetry: if agents read your logs, log hygiene is no longer just an ops expense, it's a token-budget line item.

**Still missing: the value side.** Like his earlier piece, Shpilt never asks *which* logs and metrics actually get used — the question Petkovic puts at the center of the pain cycle. Every strategy here reduces *volume*; none reduces *the right volume*. BYOC and logs-to-metrics are clever, but they optimize a signal stream whose value remains unmeasured. That's the one problem both articles, for all their tactical sharpness, leave on the table.

---

## Cross-References

- [[Reduce Logging Costs]] — The companion piece by the same author: five strategies, two of which this article repeats. This piece adds the meter-aware layer (BYOC, DPM, cardinality, logs-to-metrics, tiered levels) the earlier one only gestures at.
- [[The Observability Pain Cycle]] — The structural diagnosis these tactics can't end. Every strategy here is a trim, not a fix to the incentive loop; only BYOC touches the pricing model itself.
- [[The Three Pillars of Observability]] — Logs-to-metrics conversion is the cost-side proof of that article's claim that metrics stay far cheaper for aggregation. The "three invoices" artifact Shpilt fights against is the same one Greptime's history explains.
- [[Performance Optimization Loop]] — The shared .NET/OpenTelemetry ground: both treat measurement as a discipline, and both show a concrete OpenTelemetry interception point (SQL sanitizer there, logs-to-metrics here).

---

*Sources: [[raw/observability-cost-saving-strategies]], [[summary/observability-cost-saving-strategies]]*
*Last updated: 2026-08-26*
