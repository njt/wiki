# Your Distributed System Is Slower Than a Laptop

A sharp Cost Of Scale critique: the median company runs a seven-figure distributed streaming pipeline for workloads that a single $10,000 server could handle faster. The article revives McSherry et al.'s 2015 COST paper (Configuration that Outperforms a Single Thread), constructs a representative $1.4M/year Kafka+Flink architecture, then shows the single-machine alternative costs $57K/year — and is faster. The central argument isn't that distribution is always wrong, but that the industry stopped running the comparison a decade ago and the gap has since widened by two orders of magnitude.

---

## Key Quotes

> "The largest number in the system and the least examined."

The author points at the $875K/year platform engineering line item — 3.5 engineers maintaining a pipeline that a fraction of one engineer could replace with a single process. This is the article's real thesis: the coordination tax isn't measured in CPU cycles, it's measured in salaries. Every distributed system carries a standing army of operators who could be building things customers actually see.

> "Nobody has ever been asked in an interview to justify the cluster they did not build."

The career-incentive corollary. Operators of large distributed pipelines accumulate war stories and marketable skills. The engineer who replaces a Kafka cluster with a single Go binary has a one-line résumé entry and zero conference talks. Hebert made a similar observation in [[Queues Don't Fix Overload]] — the queue-adoption story always starts with quick wins and ends in catastrophic failure, but nobody's career suffers for the failure, only for not having the queue on their CV.

> "The difference between two machines and a distributed system is the difference between redundancy and choreography."

The sharpest line in the piece. A primary, a spare, and a switchover script get you most of the availability benefit of a distributed system without most of the operational cost. The author isn't arguing against redundancy — he's arguing against making the redundancy protocol itself a full-time engineering problem. This is the same instinct behind [[The Lindy Effect in Software]]: prefer the boring thing that works over the complex thing that might.

> "Premature scaling wears the costume of prudence."

Building for 100x actual load is treated as diligence; building for actual load is treated as risk. The arithmetic inverts: a seven-figure annual cost for headroom that customers will never observe. [[The Cost YAGNI Was Never About]] makes the same argument in economic terms — YAGNI is options pricing, and building capacity you don't need is buying an option you have no reason to exercise.

> "The laptop is still winning. The industry is still not keeping score."

The closer. McSherry et al. proved the laptop won in 2015. Hardware has improved ~100× since. The question is still missing from design reviews.

## Key Themes

- #concept — **COST (Configuration that Outperforms a Single Thread)**: the number of cores a distributed system needs before it beats a well-written single-threaded program. For several published systems, the answer was "no configuration ever catches up."
- #concept — **The coordination tax**: distributed systems spend most of their cores on coordinating work, not doing it. A single process has zero coordination overhead.
- #pattern — **The single-machine baseline**: before approving a distributed architecture, build the workload on one good server with real data. Make the distributed system justify its cost against that baseline with actual measurements.
- #concept — **Platform engineering as the hidden line item**: the largest cost in a distributed system isn't compute — it's the engineers who keep it running. This number is "the largest in the system and the least examined."
- #pattern — **Four honest reasons to distribute**: data truly exceeds one machine; availability requires replicas (but two machines ≠ a distributed system); latency requires geography (but deploy the same simple system, don't microservice it); Conway's law actually applies at your scale (rare below ~50 engineers).

## Critical Analysis

**The article is right about the problem but optimistic about the counterfactual.** The $57K alternative assumes the single-machine implementation is straightforward — that the workload decomposes cleanly, that the single process handles peak correctly, that the operator knows the domain well enough to write it without rediscovering every edge case. This is often true, but when it isn't, the distributed system's complexity isn't pure waste — it's institutional knowledge encoded in infrastructure. The real question isn't "could one machine be faster?" (yes, almost always) but "does the team have the skill to write the single-machine version?"

**The COST paper's 2015 benchmarks don't quite carry the weight the article places on them.** Twitter's follower graph of 1.5 billion edges fits in memory on a modern laptop — but that's the point. The benchmark the distributed system was designed for is now a local workload. The article's contribution is observing that this has been true for years and the industry hasn't adjusted its defaults.

**The four "structural reasons" are the article's most original contribution and the part that generalizes beyond streaming.** "Frameworks grade their own homework" applies to every benchmark-driven engineering decision. "Careers reward complexity" is an uncomfortable truth that explains far more architectural decisions than technical merit does. "Vendors sell node-hours" is the cloud's dirty secret — the business model rewards fragmentation. "Premature scaling wears the costume of prudence" is the trap that catches every senior engineer who's been burned by under-provisioning once.

**The article's remedy — a single-machine baseline — is genuinely good engineering practice, not just rhetoric.** It's a financial control applied to architecture: you can still build the distributed system, but you must first prove it beats the simple version by enough to justify its operating cost. This is what engineering organizations that don't run this comparison are missing — not the conclusion (which may still favor distribution), but the information the comparison produces about the workload's true shape.

**The weakest link is the availability argument.** A primary and spare with a switchover script is not "most of the availability benefit of a distributed system" — it's most of the *uptime* benefit, but availability means more than "the machine is on." It means the system degrades gracefully, handles partial failure, and recovers without human intervention. The article bundles these concerns into "choreography" and dismisses them, but for many organizations, the choreography *is* the value. A two-machine setup with manual switchover that works 99.9% of the time is not the same product as a distributed system with automated failover, even if both cost the same per year of uptime.

**The article works best as a diagnostic question, not a prescription.** Before you build a distributed system, ask: have you measured the single-machine baseline? If the answer is no, you're not making an architectural decision — you're following fashion. The article doesn't need to be right about the $57K number to be right about this.

---

*Sources: [[raw/your-distributed-system-slower-than-laptop]]*
*Last updated: 2026-07-11*
