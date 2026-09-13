# Predicting the Future of Distributed Systems — Colin Breck (Tesla)

A conference talk (transcribed from video into a ytx gist, with a four-part structured summary plus the full transcript) in which Colin Breck, principal engineer for energy storage at Tesla, reads the future of distributed systems through Jeff Bezos's one-way vs. two-way door framework. His thesis in one line: storage decomposed onto object storage because it became reversible, durable-execution programming models haven't been adopted because they aren't, and operationalizing AI — "really just systems engineering," he argues — may be the risk-tolerant forcing function that changes that. The talk ends on a deliberately unanswered question about whether early adopters out-compete.

---

## What It Argues

**The framework.** One-way doors (final, or expensive to reverse) demand slowing down, the right people, executive leadership, "as much information as possible"; two-way doors should be made quickly by individuals or small teams. The expensive failure is not heavyweight process on trivial decisions but the reverse — walking through a one-way door thinking it was two-way, "and now you've burdened your organization with that decision for years to come."

**Object storage is the two-way-door substrate.** The S3 API is now universal — other clouds, software, and on-prem appliances implement it — so an app depending only on Kubernetes and the S3 API "can basically move it anywhere." Lineage: Jay Kreps's "The Log" (reliable building blocks cut distributed-system implementation "from years to weeks" — Breck's gloss: "In other words, two way doors"), his own 2012 Azure system of stateless brokers over object storage (brokers scaled "instantaneously, without repartitioning data"; no Paxos), then Aurora ("the log is the database"), Snowflake, WarpStream (eventually acquired by Confluent), SlateDB, and InfluxDB — a relational database, a warehouse, a distributed log, a key-value store, and a time-series database that all separated compute from storage with object storage underneath. Above the substrate: Parquet, catalogs (Iceberg, Delta Lake, Hudi, Duck Lake announced "two days ago"), DuckDB and DataFusion as query optimization/execution as libraries, and database-per-customer/device (Cloudflare D1, MotherDuck) blurring edge and cloud. Prediction: object storage moves into transactional and operational workloads.

**Programming models are all one-way doors.** The container is a stack of undifferentiated plumbing — keys, certs, JSON, gRPC, database and Kafka clients, logging, metrics — with business logic as a thin slice, repeated across hundreds of apps that each re-solve state, durable execution, and retries; Log4J exposed the inventory problem. His wish: capabilities pushed down into infrastructure, so a patch works like an OS patch and developers "don't even need to care." The candidates — Akka and Temporal ("adopt our API and we'll manage state distribution, partial failures"), wasmCloud ("give us arbitrary code and we will execute it"), Golem (event-sourcing the WASM VM) and Unison (a new language) — all fail his test: huge investment, engineers "skeptical about giving up control," survival risk, vendor lock-in. "This is a one way door decision after one way door decision."

**AI as forcing function.** Agentic AI "looks a lot like durable actors"; AI workflows look like durable execution frameworks; MCP/RAG/A2A/ACP are "really just interfaces and protocols for systems integration." Operationalizing AI is "really just systems engineering" — and AI teams "move fast and take risks," so the new models may take hold there first, then spread to IoT, manufacturing, commerce. Caution via Spolsky's iceberg secret: easy onboarding "is not where the hard problems are" — bet on platforms that solved durable execution *before* the AI pivot (Akka's landing page now headlines "Enterprise Agentic AI"). DeepSeek's SmallPond shows AI infrastructure innovations flowing back into general distributed systems.

**The bookend.** Peter Alvaro: abstractions leak, "so make the abstractions fluid" — Breck reads this as "an invitation to find as many two way doors as possible." The future is "getting back to basics in storage and in query processing, just with the lines of abstraction drawn in different places." Closing question, unanswered: "How safe is it to just keep doing what we already know?"

## Key Quotes

> "It's perhaps even more costly to make what you think is a two way door decision when it really was a one way door decision. And now you've burdened your organization with that decision for years to come."

The asymmetry that organizes the entire talk — and the reason the object-storage and programming-model sections reach opposite verdicts.

> "The log is the database."

The Aurora paper's line, used to fuse Kreps's log essay with storage disaggregation. [[The Log — Unifying Abstraction for Real-Time Data]] records Kreps's prediction that the log becomes "a commoditized interface"; this talk is that prediction extended from replication to primary storage.

> "By using these open source libraries you basically have some of the leading database experts in the world in query optimization working for your company."

The sharpest framing of DuckDB/DataFusion as the next decomposition stage: world-class expertise consumed as a library dependency. See [[Apache DataFusion]].

> "The path to the slope of enlightenment is to recognize that S3 is an object store, not a file system." — Chris Riccomini

Stop fighting the substrate's nature; every innovation Breck catalogs follows from accepting what object storage actually is.

> "An API by any other name is still an API, unless you need to raise money for your AI startup, in which case I prefer MCP." — Chamath, quoted by Breck

A deflation of the AI-acronym boom that also, as the gist's own critique notes, deflates past the most interesting question: could MCP/A2A standardization do for agents what S3-API ubiquity did for storage?

> "In five years is the one I picked going to be dead."

The honest confession at the center of the programming-model section: five candidates named, none ranked, no method offered beyond one-way-door anxiety.

> "Abstractions are going to leak, so make the abstractions fluid." — Peter Alvaro

The talk's bookends and its most generalizable sentence: reversibility as a design goal, not an accident.

## Key Themes

- #concept — one-way vs. two-way doors as an adoption theory; disaggregation; fluid abstractions.
- #pattern — stateless brokers over object storage; log-centric design; database-per-tenant/device; infrastructure-provided service APIs ("a postgres-compatible API" instead of Postgres).
- #tool — S3 (Express One Zone, Table Buckets), Parquet, Iceberg/Delta/Hudi/Duck Lake, DuckDB, DataFusion, Akka, Temporal, wasmCloud, Golem, Unison, SmallPond.
- #person — Colin Breck, Jay Kreps, Peter Alvaro, Chris Riccomini, Joel Spolsky, Lauren Hochstein.

## An Opinionated Take

The door framework is genuinely load-bearing: it explains why storage decomposed (reversibility was *manufactured* by API ubiquity and open formats) and why programming models stalled (adoption is irreversible). But applied as a theory of adoption it flirts with tautology — whatever got adopted turns out to have been a two-way door. The falsifiable content is the mechanism, and the wiki's evidence already agrees with it: [[celld]] coordinates Durable Objects with S3 compare-and-swap, [[Graft]] replicates SQLite through object storage with no cluster, and [[SQLite is All You Need for Durable Workflows]] extends the small-embedded-database bet to agent state. The substrate thesis is not a prediction here; it is current practice.

The weakest joint is the transactional prediction. Breck expects object storage for "transactional and operational workloads" but never bridges the gap between "11 nines of durability, which boggles the mind" and "handles my transactions" — multi-object consistency, concurrency control, and OLTP-grade latency on a blob store get no mechanics, and S3 Express One Zone gets one sentence. The gist's bundled critique is sharp on this and more: egress fees and data gravity quietly turn his celebrated two-way doors into one-way doors (moving petabytes "is not a door you walk back through"), and concentration risk is invisible — if logs, warehouses, and AI infrastructure all sit on S3-class storage, S3 outages become systemic. He admires AWS using scale to "deliver innovation that others can't" without asking what a single point of failure means for everyone standing on it.

"Operationalizing AI is just systems engineering" is half right. The substrate claim lands — teams are about to re-implement job queues and workflow engines badly, and Lauren Hochstein's "we will keep building workflow engines until morale improves" is the best line in the talk. But the deflation flattens what is genuinely new. A conventional retry resends the same bytes; an LLM retry regenerates them. The wiki's Distributed Systems evidence records the consequence — a non-deterministic client breaks naive idempotency keys because regenerated tool parameters hash differently — plus model versioning, evals, and human oversight, none of which are distributed-systems problems. By Breck's own framework, misclassifying AI's hard parts as familiar two-way-door systems engineering is precisely the mistake the talk exists to warn against.

The MCP moment is the talk's most revealing evasion: the most obvious application of his own thesis — standardization creating agent portability the way S3's API created storage portability — is waved off with a joke. And the Tesla-shaped hole is real: a principal engineer on energy storage, whose thesis says operational technology should move to object storage first, offers one anecdote from 2012 at a different company. For a talk about making "effective engineering decisions," the hardest decision on the table — which programming model, if any — gets no method, only anxiety. That honesty is better than false confidence, but the closing question is left exactly where it started.

## Related Pages

- [[The Log — Unifying Abstraction for Real-Time Data]] — strengthens: Breck explicitly traces the object-storage wave to Kreps's essay, reading "from years to weeks" as "in other words, two way doors"; this talk is the commodity-interface prediction arriving in storage itself.
- [[Aurora DSQL]] — continues: Aurora's "the log is the database" starts the decomposition Breck catalogs, and DSQL's compute/commit/storage split with coordination deferred to commit is where that line has landed; [[Aurora DSQL — Murat Demirbas' Insider Review]] supplies the quantitative caveats Breck's survey omits.
- [[Durable Execution Without History Replay]] — nuances: Breck treats "durable execution" as the shared center of all five programming-model candidates without asking how any of them recover; the TCC-vs-Temporal-replay argument is exactly the engineering question hiding under his "durable actors" gloss.
- [[The Log is the Agent]] — extends: Breck says agentic AI "looks a lot like durable actors" and stops; ActiveGraph's log-as-substrate design is what you get by applying the Aurora move — the log is the database — to agents, with determinism as reconstructability rather than reproducibility.

---
*Sources: [[raw/bcf1ef513c3568f976567e1400589b63]], [[summary/bcf1ef513c3568f976567e1400589b63]]*
*Last updated: 2026-09-13*
