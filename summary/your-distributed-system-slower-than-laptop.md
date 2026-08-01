---
url: https://codegood.co/writing/your-distributed-system-is-slower-than-a-laptop
title: "Your Distributed System Is Slower Than a Laptop"
author: CodeGood (no individual byline)
date_fetched: 2026-07-11
date_published: 2026-07-05
---

This piece argues that distributed stream-processing architectures are the
default in mid-market SaaS companies not because they outperform simpler
alternatives, but because the comparison is never run. It draws on the 2015
COST paper ("Scalability! But at what COST?") by McSherry, Isard, and Murray,
which showed a single laptop thread beating multi-hundred-core graph-processing
clusters on real workloads — including a connected-components task on
Twitter's 1.5-billion-edge follower graph where a 1970s union-find algorithm on
a laptop finished 16× faster than GraphLab on 128 cores.

The article constructs a representative 55-engineer company processing 2
billion events/day and prices the conventional distributed stack — Kafka,
Flink, 128 vCPUs — at roughly $1.38M/year, with platform engineering salaries
($875K) dwarfing compute ($262K). The single-server alternative: two
$10K machines and a fraction of one engineer, about $57K/year. The distributed
system is slower, not just more expensive, because most cores burn on
coordination overhead.

Four structural reasons keep the distributed-by-default reflex alive: frameworks
benchmark only against other distributed systems, never a well-written single
thread; careers reward operating complexity, not eliminating it; cloud vendors
sell node-hours, incentivising sprawl; and premature scaling is mistaken for
prudence. The article offers four honest justifications for distribution — data
truly won't fit, availability requires replicas, latency demands geography, or
the organisation genuinely needs partitioning — but treats each as a burden to
prove with measurement, not a starting assumption.

The core remedy: build a single-machine baseline first. Run the real workload on
one competent server, make that number the threshold a proposed architecture must
beat, and treat the comparison as an ordinary financial control. The author
cites Amazon Prime Video cutting costs 90% by consolidating cloud functions,
Segment reversing its microservices migration, and Stack Overflow serving a
billion monthly page views from nine web servers at single-digit CPU
utilisation — all evidence that the laptop is still winning while the industry
keeps not keeping score.
