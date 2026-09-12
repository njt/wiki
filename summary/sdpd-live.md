---
url: https://sdpd.live
title: "SDPD — Systems Design Police Department"
author: Unknown (sdpd.live)
date_fetched: 2026-06-15
date_published: Unknown
capture_method: browser automation (surf)
topics:
  - distributed-systems
---

# SDPD — Systems Design Police Department

The city of Distributia runs on distributed systems. Things keep breaking. Diagnose failures. Design solutions. Solve cases.

ROOKIE RANK — 0/33 CASES — 0% CLEARED

## Case Categories

### Replication & Redundancy (0/3)
1. The Single Point of Failure — Criminal records go dark across all precincts
2. The Stale Read — An officer arrests someone on a dismissed warrant
3. The Lost Evidence — 16 crime scene photos vanish into thin air

### Consistency & Consensus (0/5)
4. The Split Brain — Two precincts, two truths, one broken network
5. The Phantom Vote — Evidence room lockdown fails as multiple nodes claim leadership
6. The Conflicting Orders — Two leaders, two assignments, one very confused officer
7. The Time Traveler's Dilemma — Evidence timestamps defy the laws of physics
8. The Byzantine Witness — A rogue forensics node lies to everyone — differently

### Load Balancing & Scaling (0/5)
9. The Overwhelmed Gateway — City-wide emergency brings the single gateway to its knees
10. The Thundering Herd — Every precinct stampedes the database at the same moment
11. The Hot Partition — One shard drowns while the others sit idle
12. The Vertical Limit — Crime analytics server hits its hardware ceiling
13. The Sticky Situation — Server crash wipes out hundreds of active officer sessions

### Caching (0/4)
14. The Ghost Record — A deleted criminal record keeps haunting the system
15. The Cache Avalanche — All cache entries expire simultaneously, crushing the database
16. The Dog Pile — Hundreds of requests stampede to rebuild a single cache entry
17. The Stale Menu — CDN serves outdated Most Wanted list hours after update

### Messaging & Queues (0/4)
18. The Lost Dispatch — Server crash loses all pending dispatch calls from the queue
19. The Double Arrest — Duplicate message delivery causes the same warrant to be processed twice
20. The Backed Up Pipeline — Dispatch queue overflows as consumer service crashes silently
21. The Out of Order — Crime reports arrive jumbled as parallel consumers scramble the sequence

### Distributed Storage (0/4)
22. The Missing Shard — An entire letter range of criminals vanishes from the database
23. The Inconsistent Lineup — Two precincts see different versions of the same suspect at the same time
24. The Corrupted Archive — Evidence arrives corrupted but nobody knows where the damage happened
25. The Overloaded Vault — Write-heavy evidence logging causes massive storage amplification

### Network & Communication (0/4)
26. The Timeout Trap — A slow downstream service triggers cascading timeouts across the entire system
27. The Retry Storm — One failed request becomes a million knocking at the door
28. The Circuit Breaker — One fallen domino topples the entire dispatch center
29. The DNS Disaster — When a server moves and nobody gets the new address

### Advanced Patterns (0/4)
30. The Deadlock District — Two services locked in an eternal standoff
31. The Phantom Read — The crime statistics that changed mid-count
32. The Saga Failure — A booking process that forgot how to undo its mistakes
33. The Rate Limiter — One integration floods the system, drowning everyone else
