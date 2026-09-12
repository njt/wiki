# Meerkat — QuePaxa Consensus at Cloudflare

Cloudflare Research's internal consensus service, powered by the QuePaxa algorithm (Tennage & Băsescu et al., 2023), which lets *all* replicas perform writes at all times with no leader-elected timeout—a deliberate departure from Raft's single-leader bottleneck. Designed for small control-plane state across 330+ global data centers, currently in proof-of-concept with up to 50 replicas. Not yet production, but already demonstrating that clusters survive constant leader failures with zero error-rate increase.

---

## Key Architecture

Meerkat translates app-specific requests into a replicated log of "slots." All replicas maintain the same log. The critical invariant: if any two replicas decide on a value for a slot, those values are identical. A replica that missed a write cannot complete a subsequent read without first learning about the already-decided slot—this is what delivers linearizability without a single-writer bottleneck.

**Leader-optional consensus.** QuePaxa has a leader, but it's optional. The only advantage of proposing through the leader is fewer round trips (1 vs. 3+). Clients can contact any replica—or multiple replicas concurrently for the same proposal—without destructive interference. "Replicas *work together* to decide one of the proposed values."

**Failure model.** Remains available as long as a majority of replicas are alive and a client can contact any member of that majority. No Byzantine fault tolerance.

## Key Quotes

> "All replicas can perform writes at all times, and progress is never halted due to a timeout."

This is the headline claim. Raft's fundamental limitation is that leader failure → timeout → election → unavailability window. QuePaxa eliminates that window entirely. The tradeoff is 3x round trips when writing through a non-leader, which they mitigate with batching, stale reads, and transaction bundling.

> "We've experienced multiple incidents caused by unavailable leaders in consensus-driven systems."

Cloudflare is not theorizing here—this is scar tissue. The blog post is unusually candid about the operational pain that motivated the research investment.

> "In these clusters leaders constantly fail, yet the cluster keeps operating with no increase in error-rate."

The empirical result from their 50-replica global PoC. Leaders failing constantly with zero impact is the strongest possible validation of the architecture.

## Performance Tradeoffs

QuePaxa takes 1–3 round trips per proposal. In a globally distributed system, each round trip is tens to hundreds of milliseconds. The mitigations are pragmatic:

- **Colocate replicas** when latency matters more than fault domain diversity
- **Batch writes** (10 writes in 10ms → 1 proposal)
- **Stale reads** from local replica state (never inconsistent, just not latest)
- **Bundle operations** into transactions (compare-and-swap, general transactions)

This makes Meerkat suited for "control plane information that is written infrequently but must remain consistent"—not for high-throughput data paths.

## Critical Analysis

**This is the consensus algorithm the distributed systems literature has been promising but not shipping.** Leaderless Paxos variants (EPaxos, Generalized Paxos, etc.) exist in papers but rarely in production systems. QuePaxa appears to have made it through the valley of death from publication to implementation, and the Cloudflare blog post format—detailed but not academic—suggests they're serious about shipping it, not just publishing about it.

**The "optional leader" framing is the right pitch.** It doesn't claim leaderlessness—QuePaxa has a leader. It claims the leader is an optimization, not a dependency. This is both technically honest and operationally meaningful: if the leader dies, nothing breaks, writes just get 3x slower until a new leader naturally emerges. That's a dramatically better failure mode than Raft's hard unavailability window.

**The round-trip cost is real but strategically acceptable.** 1–3 round trips to a quorum means Meerkat's latency floor is set by the speed of light between data centers. For control-plane state—configuration, leadership elections for other systems, feature flags—this is fine. It would be catastrophic for a caching layer or a hot data path. Cloudflare knows this and is explicit about the scope.

**Rust + formal verification + deterministic simulation testing is the trustworthy-systems trifecta.** The future-work list (formal verification of the Rust implementation, deterministic simulation for bug discovery, peer-reviewed manuscript) signals that Cloudflare understands consensus is a correctness-critical component. You don't ship a consensus system on vibes. The ambition to formally verify parts of the implementation is notably rare outside of AWS (who did this for S3's consensus layer).

**The omission of Byzantine fault tolerance is notable but correct.** BFT adds significant complexity and latency for a threat model that doesn't apply to Cloudflare's internal control plane. If an attacker can compromise your internal consensus replicas, you have bigger problems than consensus correctness.

**What's not in the post matters.** No latency numbers. No throughput benchmarks. No comparison to etcd's Raft implementation at scale. No discussion of how Meerkat handles membership changes (adding/removing replicas from a running cluster). These are the hard parts of any consensus system in production, and their absence suggests Meerkat is still in the research-to-engineering transition.

**The connection to agent orchestration is real and underexplored.** The [[Distributed Systems]] hub page notes that "consensus protocols for agents" is a missing piece—how do multiple agents agree on shared state? Meerkat is a general-purpose consensus service. If Cloudflare eventually exposes it externally, it could become the consensus primitive that agent orchestration frameworks currently lack. But that's speculative; for now, it's an internal infrastructure piece.

**Bottom line:** A credible, well-scoped consensus system from a company that runs one of the largest distributed infrastructures on the planet. The operational motivation is honest, the architecture is sound, and the "no leader dependency" property addresses a real failure mode that Raft-based systems suffer in production. Worth tracking as it moves toward production deployment.

---

## Key Themes

#consensus #distributed-systems #Cloudflare #Raft #QuePaxa #linearizability #tool

## Related Pages

- [[Distributed Systems]] — Hub page; explicitly notes consensus protocols as a missing piece in agent orchestration
- [[SDPD — Systems Design Police Department]] — The "Split Brain," "Phantom Vote," and "Conflicting Orders" cases are exactly the failures Meerkat is designed to avoid
- [[21 Years and Counting of Eight Fallacies of Distributed Computing]] — The fallacies that make global consensus hard
- [[Orchestrating AI Code Review at Scale]] — Another Cloudflare infrastructure piece; evidence of their internal platform investment pattern
- [[Postgres Transactions Are a Distributed Systems Superpower]] — The alternative approach: co-locate state and use database transactions instead of consensus

---

*Sources: [[raw/meerkat-introduction]]*
*Last updated: 2026-07-11*
