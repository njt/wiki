# SDPD — Systems Design Police Department

A gamified, interactive learning site that teaches distributed systems through the frame of a police detective solving cases. "SDPD" stands for Systems Design Police Department, set in the fictional city of Distributia — a city that "runs on distributed systems." You play a rookie detective working through 33 cases, each of which is a classic distributed-systems failure scenario dressed as a crime to solve. The site is free, web-based, and structured as a progressive unlock: solve earlier cases to access later ones.

---

## Case Taxonomy

The 33 cases are grouped into 8 categories. The case names map directly to canonical distributed-systems failure modes — the mapping is unambiguous and deliberate.

### Replication & Redundancy (3 cases)
1. **The Single Point of Failure** — Criminal records go dark across all precincts. The classic SPOF: one component whose failure takes down the system.
2. **The Stale Read** — Officer arrests someone on a dismissed warrant. Read-after-write consistency failure in a replicated database.
3. **The Lost Evidence** — 16 crime scene photos vanish. Data loss from insufficient replication factor.

### Consistency & Consensus (5 cases)
4. **The Split Brain** — Two precincts, two truths, one broken network. The classic network partition scenario where both sides believe they're the leader.
5. **The Phantom Vote** — Evidence room lockdown fails as multiple nodes claim leadership. Leader election gone wrong.
6. **The Conflicting Orders** — Two leaders, two assignments, one confused officer. Dual-master conflict resolution failure.
7. **The Time Traveler's Dilemma** — Evidence timestamps defy physics. Clock skew and the unreliability of wall-clock time in distributed systems.
8. **The Byzantine Witness** — A rogue forensics node lies to everyone, differently. The Byzantine Generals Problem: a node sending conflicting information to different peers.

### Load Balancing & Scaling (5 cases)
9. **The Overwhelmed Gateway** — City-wide emergency brings the single gateway to its knees. The single ingress point bottleneck.
10. **The Thundering Herd** — Every precinct stampedes the database at the same moment. Many clients waking simultaneously and hammering a shared resource.
11. **The Hot Partition** — One shard drowns while others sit idle. Uneven data distribution across shards.
12. **The Vertical Limit** — Crime analytics server hits its hardware ceiling. The limits of vertical scaling.
13. **The Sticky Situation** — Server crash wipes out hundreds of active sessions. Sticky session state lost when a server fails.

### Caching (4 cases)
14. **The Ghost Record** — A deleted record keeps haunting the system. Stale cache entries surviving their source data.
15. **The Cache Avalanche** — All cache entries expire simultaneously, crushing the database. Synchronized TTL expiry triggering a load spike.
16. **The Dog Pile** — Hundreds of requests stampede to rebuild a single cache entry. Cache stampede on a cold or expired key.
17. **The Stale Menu** — CDN serves outdated Most Wanted list hours after update. CDN cache invalidation lag.

### Messaging & Queues (4 cases)
18. **The Lost Dispatch** — Server crash loses all pending dispatch calls. At-most-once delivery without persistence.
19. **The Double Arrest** — Same warrant processed twice. At-least-once delivery without idempotency.
20. **The Backed Up Pipeline** — Dispatch queue overflows as consumer crashes silently. Backpressure failure from an unmonitored dead consumer.
21. **The Out of Order** — Crime reports arrive jumbled. Parallel consumers breaking message ordering guarantees.

### Distributed Storage (4 cases)
22. **The Missing Shard** — An entire letter range of criminals vanishes. Shard mapping or routing failure.
23. **The Inconsistent Lineup** — Two precincts see different versions of the same suspect. Replication lag causing inconsistent reads.
24. **The Corrupted Archive** — Evidence arrives corrupted but nobody knows where. Silent data corruption without checksums.
25. **The Overloaded Vault** — Write-heavy logging causes massive storage amplification. Write amplification in log-structured or copy-on-write storage.

### Network & Communication (4 cases)
26. **The Timeout Trap** — A slow downstream service triggers cascading timeouts. Timeout misconfiguration causing cascade failure.
27. **The Retry Storm** — One failed request becomes a million. Unbounded retry amplification.
28. **The Circuit Breaker** — One fallen domino topples the entire dispatch center. Cascading failure from a missing circuit breaker.
29. **The DNS Disaster** — Server moves and nobody gets the new address. Stale DNS caching (TTL ignored or too long).

### Advanced Patterns (4 cases)
30. **The Deadlock District** — Two services locked in an eternal standoff. Classic deadlock between distributed services.
31. **The Phantom Read** — Crime statistics that changed mid-count. Non-repeatable read / read skew.
32. **The Saga Failure** — A booking process that forgot how to undo its mistakes. Missing compensating transactions in a distributed saga.
33. **The Rate Limiter** — One integration floods the system, drowning everyone else. Noisy-neighbor problem without rate limiting.

---

## Key Themes

#learning #distributed-systems #tool #education #gamification

---

## Critical Analysis

**This is a terrific teaching tool.** The core insight — that diagnosing distributed systems failures IS detective work — is not just a cute framing device. It's pedagogically correct. You arrive at a symptom (the system is broken), you interview witnesses (check logs and metrics), you form hypotheses about which component failed and how, and you test them. The detective metaphor maps naturally onto the debugging workflow that every SRE and backend engineer already performs.

**The taxonomy is more valuable than the individual cases.** Even without playing through a single case, the list of 33 failure modes organized into 8 categories is a useful diagnostic checklist. When your production system is broken, running down this list — "is it a split brain? a cache avalanche? a retry storm?" — is a more structured approach than the "stare at dashboards until something looks wrong" method most teams use. The SDPD list is practically a differential diagnosis guide for distributed systems.

**How does this compare to reading papers?** Papers give you depth on one problem. The Paxos paper teaches you consensus. The Dynamo paper teaches you eventual consistency. SDPD gives you breadth — 33 failure modes, one per "case" — with enough context to recognize each pattern when you see it in the wild. It's a survey course, not a seminar. For someone new to distributed systems, starting with SDPD and THEN reading the canonical papers is probably the right order: build the mental map first, then fill in the details.

**How does this compare to the eight fallacies?** The [[21 Years and Counting of Eight Fallacies of Distributed Computing|eight fallacies]] are principles — *why* things break. SDPD's cases are instantiations — *what* the breakage looks like. Fallacy #1 (the network is reliable) manifests as The Timeout Trap, The Retry Storm, The Circuit Breaker, and The DNS Disaster. Fallacy #2 (latency is zero) manifests as The Stale Read and The Thundering Herd. The two resources are complementary: the fallacies give you mental models, SDPD gives you diagnostic patterns.

**Limitations.** The site is classification-focused — you diagnose the problem, not fix it. Knowing a system has a split brain is half the battle; the other half is knowing whether to use a fencing token, a witness node, or majority quorum. The site doesn't teach solutions. It also doesn't cover the operational side: how do you detect these failures in production? What metrics should you monitor? What alarms should you set? These are gaps that reading papers and operating real systems fills.

**The gamification is smart but thin.** The "rookie detective" framing and progressive case unlocking adds motivation, but the actual interaction appears to be multiple-choice diagnosis. A truly interactive version — where you could inject failures into a simulated distributed system and observe the effects — would be a more powerful learning tool. But that's a much harder engineering problem, and SDPD's value-to-effort ratio as a web quiz is still high.

**Bottom line:** If you work on distributed systems, bookmark this. It's a better taxonomy of failure modes than most textbooks offer, and the detective framing makes it stick.

---

## Related Pages

- [[Distributed Systems]] — Hub page: agent orchestration IS distributed systems, and this taxonomy applies to agent failures too
- [[21 Years and Counting of Eight Fallacies of Distributed Computing]] — The principles these failure modes instantiate
- [[Queues Don't Fix Overload]] — Hebert's argument that buffering hides constraints; directly relevant to the Messaging & Queues and Load Balancing categories
- [[Process-Based Concurrency BEAM OTP]] — The BEAM VM was designed to handle many of these failure modes at the language runtime level

---

*Source: [[raw/sdpd-live]], captured via browser automation (surf) from [sdpd.live](https://sdpd.live/)*
*Last updated: 2026-06-15*
