# ClickBench Playground

Alexey Milovidov's engineering deep-dive on hosting ~100 database systems — from ClickHouse to BQN — in an interactive playground where anyone can query, compare, and race them. The post is three things in one: a benchmark origin story, an infrastructure war story (Firecracker VMs, BtrFS compression tricks, elastic memory via swap-to-page-cache), and a meditation on what you learn when you put a hundred databases behind the same interface and start asking "what if?"

---

## Key Quotes

> "I wanted to test as many databases as I could, so the focus was to make adding a new database easier. Every database was represented as a directory with a couple of shell scripts."

The ClickBench origin is almost comically low-ceremony. No Docker Compose, no Kubernetes, no CI pipeline — just shell scripts on EC2. This minimalism turned out to be the feature: it lowered the contribution barrier enough that ClickBench became the most popular open benchmark for analytical databases. Compare the over-engineered alternatives that nobody contributes to because the onboarding cost exceeds the motivation.

> "Some contenders were cheating on the benchmark by 'forgetting' to flush the page cache between queries, or not including some optimization work into the loading time."

Benchmark cheating as a distributed social phenomenon: contributors catch each other. The mechanism is self-policing open source — every cheater is caught by a competitor who runs the same queries and gets different results. This is a more interesting version of the benchmark exploitation problem in [[Benchmark Exploitation]]: here the integrity mechanism isn't technical (no pytest hooks to force passes) but social — the adversarial incentive of competitors who benefit from exposing cheating.

> "Refactoring a hundred shell scripts is well beyond human capabilities, and I tried to do it multiple times - first manually, then with AI. After a few tries, it was done."

A throwaway line that's actually a significant data point about AI-assisted code transformation. A hundred shell scripts, each slightly different, each doing the same logical operations — this is exactly the kind of refactoring that AI should excel at (pattern recognition + mechanical transformation), and Milovidov confirms it worked after "a few tries." No details on which AI or how, but the existence proof matters.

> "We are building a cloud inside a cloud."

The cleanest one-line summary of the architecture: Firecracker microVMs running on AWS bare metal. Every solution Milovidov considered and rejected (per-system EC2 at $536K/year, on-demand machines that bots would spin up, Lambda's size limits and runaway costs, ECS/EKS Docker-in-Docker nightmares, Kubernetes because "some friends love Kubernetes because they are in an abusive relationship with its complexity") is a tour of why "just run it in the cloud" isn't a helpful answer for this class of problem.

> "The swap space of the guest machine is mapped to the page cache of the host machine, and as long as the total memory space of the host machine is not contended, the guest machine will work as fast as if it had a larger memory amount."

The most delightful hack in the article. Guest swap → virtual disk → host page cache (fsync disabled). Memory becomes elastic, like CPU. When contention spikes, the host OOM killer sorts it out. This is the kind of trick you only discover when you're deep enough into a problem to understand which layers are real boundaries and which are conventions you can invert.

> "I didn't trust BtrFS because I thought it was some sort of parody attempt to reimplement ZFS, but worse. But it supports both compression and reflinks, so I didn't have much choice."

Honest about the emotional dimension of engineering choices. XFS (reflinks, no compression) vs. BtrFS (both, but bad reputation). The constraint forced the choice, and the choice worked — after fixing the compressibility heuristic with `compress-force=zstd:6` and manual defragmentation. Sometimes the tool you distrust is the one that solves your problem.

> "I don't trust any database that requires Docker to run. Good databases, like ClickHouse, work fine on any system. But I still wanted to support all the other, not-so-good databases."

The ClickHouse creator's version of "it's not a bug, it's a feature" — delivered with enough self-awareness to be charming rather than smug.

> "I created this service for myself. I love to collect various database systems, production and experimental, popular and obscure, even weird and crackpot databases."

The post's real thesis, revealed at the end. This wasn't a product launch or a marketing exercise. It's a collector building a display case.

---

## Key Themes

#databases #benchmarking #virtualization #infrastructure #ClickHouse #filesystems #engineering

- **#databases** — The playground spans relational (Postgres, MySQL, ClickHouse), dataframe (Pandas, Polars), document (Mongo, Elastic), array (BQN), and embedded engines (DuckDB, DataFusion). It's the most comprehensive interactive comparison of database query languages in existence.
- **#benchmarking** — ClickBench's evolution from ClickHouse-specific (2013) to open (2022) to interactive playground (2026). The common-interface refactoring unlocked experiments that were impossible with shell scripts: larger datasets, more queries, concurrent QPS, variable hardware. Benchmark integrity enforced socially (competitors catch cheaters) and mechanically (restart before cold queries).
- **#virtualization** — The full stack: nested virtualization on AWS bare metal, Firecracker microVMs, lazy memory allocation, elastic swap, watchdog CPU limits, SNI-filtering proxy for selective internet access, snapshot-based reset after errors. A production reference architecture for hosting many untrusted systems on one machine. Connects to the isolation patterns in [[How We Contain Claude]] and the Firecracker journey in [[Nango — Running Untrusted Customer Code at Scale]].
- **#filesystems** — The storage pipeline is a masterclass in commodity-hardware optimization: sparse files (`truncate -s 200G`, `init_on_free=1`), ext4 → XFS for reflinks (copy-on-write to avoid duplication between snapshot and runtime), XFS → BtrFS for compression (zstd:6), `sync + drop_caches + fstrim` before snapshot, `cp --sparse=always` to preserve holes. All hundred systems fit in 7.5 TB. This is the kind of deep filesystem knowledge that's becoming rare as the industry abstracts storage behind cloud APIs.

---

## Critical Analysis

**The article is three pieces fighting to be one.** The ClickBench origin story is useful context. The infrastructure war story (Firecracker, BtrFS, the proxy, the networking) is the best part and could stand alone as a systems engineering classic. The "fun things" section (query language comparisons, 10× dataset experiments) is fascinating but underdeveloped — Milovidov teases findings ("some top entrants were over-optimizing," "some systems stop working with any reasonable parallelism") without naming names or showing data. The post would be stronger if one of these three threads were cut, or if they were published as separate pieces with more depth on each.

**What's missing: the data.** Milovidov ran 10× datasets, 100-query benchmarks, concurrent QPS tests, and hardware-size sweeps — and shares almost none of the results. "I found that some of the top entrants were over-optimizing for the default queries" is a claim that demands evidence. The playground exists; the benchmark results presumably exist. The gap between "I ran these experiments" and "here's what I found" is the difference between an engineering blog post and a research contribution. Given Milovidov's incentives (he works at ClickHouse, and ClickHouse tops the results), the omission might be strategic — or it might just be that the post is already 3,000 words and something had to give.

**The "I" voice is effective.** Milovidov writes in first person throughout, and it works. The emotional honesty — distrusting BtrFS, loving to collect databases, finding someone who shares the obsession — makes the technical content more readable. Compare to the anonymous corporate "we" that drains personality from most vendor engineering blogs.

**The Docker-in-Docker problem is a real indictment.** Milovidov's complaint about databases that require Docker ("I don't trust any database that requires Docker to run") is funny but also substantive. Docker inside a Firecracker VM required disabling iptables, loading extra kernel modules, and fighting the guest's networking layer. This is the operational tax of the container-everything era: every layer of abstraction adds a layer of interference. [[Lean Software Production]]'s "eliminate waste" principle applies here — Docker for a single-process database is waste that compounds when you try to nest it.

**Connection to [[Why Are Databases So Hard]].** gtowey's article argues databases are hard because of physics (the speed of light). Milovidov shows another dimension: databases are hard to *host* because they have wildly different operational requirements. Some need 32 GB RAM. Some need obscure JVM versions. Some require Docker. Some crash, OOM, infinite-loop, or eat disk. "There is one such system that you can configure this way - it is ClickHouse, but I don't know any other" is both a flex and a genuine insight: most databases aren't production-grade in the sense of running unattended for long periods. The playground's solution — reset from snapshot after any error — is an admission that you can't fix the databases; you can only contain the damage.

**The laconic Kubernetes joke is telling.** "I know some friends who love Kubernetes because they are in an abusive relationship with its complexity" is the kind of aside that only works because Milovidov spent paragraphs explaining *exactly* why Kubernetes didn't fit (Docker-in-Docker, no CRIU snapshot support, cost equivalent to EC2). He did the work and then made the joke, rather than making the joke instead of doing the work. The right way to be dismissive.

**Bottom line.** This is one of the best infrastructure engineering posts of 2026. The storage section alone — sparse files, reflinks, compression, `fstrim`, `cp --sparse=always` — is a practical education in Linux filesystems that most cloud-native engineers never need to learn. The architectural honesty (here's what I tried, here's why it didn't work, here's what I settled on) is a model for technical writing. The missing benchmark data is the only real flaw, and it's a significant one — the post promises more than it delivers on the "what did we learn about databases" front.

---

*Sources: [[raw/clickbench-playground]], [[summary/clickbench-playground]]*
*Last updated: 2026-08-06*
