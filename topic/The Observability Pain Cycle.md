# The Observability Pain Cycle

Nikola Petkovic's structural diagnosis of why observability bills keep blowing up no matter how many times you trim them. The store-everything-up-front pricing model mainstream observability vendors were built on makes noise the default, excludes the one person who knows which signals matter, and leaves the invoice as the only feedback signal — guaranteeing a boom-bust cycle that no one-time cleanup can end.

---

## Key Quotes

> The vendor asks you to ingest everything up front, and charge you the moment this huge volume of telemetry is ingested. Only then do you get a chance to explore that data and get some value from a small portion of it.

The setup in one sentence: payment is decoupled from value in time. You pay at full height from the first minute, and value arrives later — if at all — on a small fraction of the data. The spend curve is flat; the value curve is a spike.

> It is always the pain of a high bill that triggers the trimming exercise.

The mechanism distilled. The loop's trigger isn't "we found waste" but "we felt pain," and that single fact determines everything downstream: trims are reactive, coarse, and stop the moment the pain subsides, not when the waste is gone.

> An average vanilla Kubernetes cluster produces almost 100,000 time series out of the box.

The concrete scale of the noise problem. Auto-instrumentation and built-in library metrics made emission free, so the default volume is enormous before anyone writes a single dashboard. This is why "you never know which signal you'll need" is a thin-slice argument, not a general one.

> The classic cost controls decide how much data to keep, not which data matters.

The sharpest technical distinction in the piece. Sampling, cardinality pruning, and usage caps all trim volume without knowing value. The one exception — tail-based trace sampling — works only because traces carry a value marker (latency and error status) the pipeline can read. Metrics and logs have no such marker; their value lives downstream in what actually uses them, and no pipeline looks there.

> The one party who actually knows what is valuable has a little say in the loop.

The core grievance, and the article's real subject: broken incentive alignment. The vendor's defaults decide what gets collected, the bill decides what gets cut, and the user — who knows their own systems and which questions they need answered — is a spectator to their own spend.

> This will not fix itself, and I doubt it will be fixed by those who profit from the noise.

The coda. AI-era telemetry — agent and AI workloads emitting faster than any human team — makes the cycle worse, and the author refuses to expect the people whose margin depends on noise to solve it.

## Key Themes

- **#pattern The Observability Pain Cycle** — a named boom-bust loop: store everything up front → noise accumulates → invoice crosses the pain threshold → coarse one-time trim → filters never re-tuned → volume regrows. The correcting signal is late (already paid for) and coarse (uncorrelated with value).
- **#concept Store-everything-up-front** — the founding pricing philosophy of mainstream observability, reasonable when telemetry was hand-instrumented and low-volume, never revisited once cloud, containers, and auto-instrumentation made emission free. It "protects the vendor's interest."
- **#concept Value is downstream, not in the signal** — traces carry a value marker the pipeline can read (latency/error); metrics and logs don't. Their value is defined by whether anything downstream consumes them — exactly the information pipelines never inspect.
- **#tool Observability vendor economics** — the tactics that preserve the status quo: "you never know the next incident," "storage is cheap," AI features downstream of collection, and transparent billing that itemizes without answering which signals earn their keep.
- **#person Nikola Petkovic** — the author, writing from the customer's side of the vendor relationship.

## Critical Analysis

**The diagnosis is the contribution; the remedy is underdeveloped.** Petkovic names a real loop and explains *why* it recurs — a structural incentive mismatch, not engineer laziness — which is more than most cost-cutting posts do. But the positive program amounts to "you should be the one deciding which questions your observability answers." That's an aspiration, not a mechanism. The hard problem — how to make "which data matters" a decision the pipeline can actually see — is gestured at with tail-based sampling and then left on the table.

**It's the missing second half of [[Reduce Logging Costs]].** Shpilt gives the tactics (sampling, tiered storage, in-code cleanup, killing INFO logs); Petkovic explains why those tactics are one-time and why the bill keeps coming back. Read together they form a coherent account: Shpilt's "reduce before it leaves the application" treats the symptom, Petkovic's cycle is the reason it keeps recurring.

**The tail-based-sampling exception is the load-bearing insight.** It's the one place the article shows a *working* answer to its own question, and it does so by pointing at the structural difference between signal types: traces self-describe their value, metrics and logs don't. The implied research direction — find the value marker for metrics and logs, i.e. measure downstream usage — is the most promising thread in the piece, and the author underlines it rather than pursuing it.

**The cynical ending is earned but stops short.** "It won't be fixed by those who profit from the noise" is fair, but it dodges whether vendor incentives *could* realign — pricing on value, sampling-by-default with on-demand expansion, or platforms that close the loop by tracking which series actually feed a panel or alert. The article is better at naming the disease than imagining the cure.

**The AI-era coda is more than a throwaway.** Agent and AI workloads don't just add telemetry — they emit it automatically, at a rate no human team can keep pace with, so the volume grows without anyone choosing to instrument. That's store-everything-up-front's endgame: emission is free, so the noise compounds interest on itself.

## Cross-References

- [[Reduce Logging Costs]] — The tactical complement: Shpilt's five strategies are how you trim; Petkovic's cycle is why the trim doesn't stick. One treats the symptom, the other names the disease.
- [[The Three Pillars of Observability]] — The same vendor economics from the history side: "three separate invoices" as a pricing artifact, pillars persisting partly because unifying meant incumbents giving up pricing power. Petkovic is the customer-side account of that same incentive.
- [[Queues Don't Fix Overload]] — The shared structural lens: treat causes, not symptoms. The cycle recurs because the cause — incentive misalignment plus filters nobody re-tunes — is never addressed, only the symptom (the bill).
- [[Lean, Not Backpressure]] — The lean manufacturing reading: Petkovic's "the one party who knows what's valuable has no say" is quality-upstream inverted — the person with the knowledge is excluded from the decision.

---
*Sources: [[raw/observability-pain-cycle]], [[summary/observability-pain-cycle]]*
*Last updated: 2026-08-25*
