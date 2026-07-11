---
url: https://codegood.co/writing/your-distributed-system-is-slower-than-a-laptop
title: Your Distributed System Is Slower Than a Laptop
author: CodeGood (no individual byline)
date_fetched: 2026-07-11
date_published: 2026-07-05
site: codegood.co
---

# Your Distributed System Is Slower Than a Laptop

**Published:** 5 July 2026
**Author:** CodeGood (no individual byline provided)

## Summary of the Article

The piece revisits a landmark 2015 paper by Frank McSherry, Michael Isard, and Derek Murray titled "Scalability! But at what COST?" The researchers took benchmarks from leading distributed graph-processing systems and ran the same workloads on a single thread of a laptop — and won.

The paper introduced COST: the Configuration that Outperforms a Single Thread. For several published systems, the answer was in the hundreds of cores. For some, "no configuration of any size ever caught up." One task — finding connected components in Twitter's follower graph of 1.5 billion relationships — took GraphLab 242 seconds using 128 cores, while a laptop running a 1970s-era union-find algorithm finished in 15 seconds. Similarly, a single thread completed 20 PageRank iterations in 300 seconds, while GraphX needed 128 cores to take 419.

## A $1.4M System Doing a $57K Job

The article constructs a representative mid-market SaaS company: 55 engineers, 2 billion events per day (23,000/sec), 1 TB daily volume (12 MB/sec). The conventional architecture: a 9-machine Kafka cluster, a Flink deployment of 128 virtual CPUs, and supporting services. The cost breakdown:

- **Compute & network:** $262,000/year — 128 vCPUs of stream processing at $0.10/core-hour, Kafka machines with replication, and cross-zone data transfer charges.
- **Platform engineering:** $875,000/year — 3.5 engineers at $250,000 loaded cost each. The author notes this is "the largest number in the system and the least examined."
- **Incidents:** $240,000/year — one serious incident per quarter at $60,000.
- **Total: $1,377,000/year**

The alternative: two servers (primary + spare) at $10,000 each, spread over three years, plus a fifth of one engineer. That comes to roughly $57,000/year — and the single process is faster, not merely cheaper. The distributed system spends most of its cores on coordination overhead rather than the actual business problem.

## Why the Reflex Persists

The article traces the distributed-by-default habit to the 2004–2012 era when it often made sense. Google's MapReduce paper described splitting jobs across thousands of cheap machines because a commodity server of that time had 4 cores and 8 GB of RAM. Data simply didn't fit on one machine.

Since then, every relevant metric has improved by factors of 100 or more. A single server now offers 192 cores; cloud machines reach 24 TB of memory; SSDs read 14 GB/s. A $5,000 workstation exceeds the capability of the clusters MapReduce was designed for. Yet the industry still builds for the workloads of a few giants while selling to hundreds of thousands of smaller customers.

The article cites real-world validations: Amazon's Prime Video team reduced infrastructure cost by 90% by merging scattered cloud functions into one program. Segment reversed its microservices migration. Stack Overflow served over a billion monthly page views from "one program running on nine web servers, processors idling in single digits."

## Why the Comparison Never Gets Run

Four structural reasons:

1. **Frameworks grade their own homework** — distributed systems are benchmarked against other distributed systems, not against a well-written single-threaded program. The framework often forces suboptimal algorithms, then takes credit for scaling them.

2. **Careers reward complexity** — operators of large distributed pipelines build CVs; someone who simplified it all away has an anecdote. "Nobody has ever been asked in an interview to justify the cluster they did not build."

3. **Vendors sell node-hours** — consolidating 64 cores onto one machine destroys cloud revenue. "The commercial gravity of the cloud... pulls toward many-machine designs."

4. **Premature scaling wears the costume of prudence** — building for 100x actual load is treated as diligence; building for actual load is treated as risk. The arithmetic shows the opposite: a seven-figure annual cost with nothing customers can see.

## Four Honest Reasons to Distribute

The article argues distribution is a cost to justify by measurement, not a default. Four valid justifications:

1. **The data does not fit** — but "does not fit" means truly exceeds 24 TB of memory or a shelf of fast drives, not "does not fit the default configuration."

2. **Availability requires replicas** — a primary, a spare, and a switchover script buy most availability benefits. "The difference between two machines and a distributed system is the difference between redundancy and choreography."

3. **Latency requires geography** — the speed of light justifies deploying the same simple system in multiple regions, not breaking it into microservices.

4. **The organisation needs partitioning** — Conway's law is real, but routinely invoked at a tenth of the scale where it applies. A 30-engineer company running 40 microservices "has not partitioned its organisation. It has fractured it."

## The Remedy

Before any distributed architecture is approved, build the single-machine baseline: the workload implemented competently on one good server, with real data volumes and real access patterns. Make that number the threshold the proposed architecture must beat by a margin sufficient to justify its operating cost. The author calls this "an ordinary financial control applied to a category of spending that has escaped it."

Teams that build the baseline learn their workload's true shape — the dataset that fits in memory, the peak that's a tenth of the estimate, the one dominant query. Most will find that the comparison nobody ran was the better system all along: faster, cheaper, and comprehensible to the people on call.

McSherry and co-authors suggested measuring new systems against the hardware on an engineer's desk. A decade later, that hardware is a hundred times better, and the question is still missing from most design reviews. "The laptop is still winning. The industry is still not keeping score."
