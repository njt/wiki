# Figma's Nimbus — Sandboxes as a Platform Contract

Figma's engineering team explains Nimbus, the centralized sandbox platform that backs Figma Make and the design agent: tens of thousands of sandboxes running untrusted code, provisioned in under a second via warm pools, with a read/write-split egress policy and a simulator-tested control loop. It's a picture of what sandboxing looks like when it stops being a per-project decision and becomes shared platform infrastructure.

---

Nimbus's core move is a contract: the **namespace**. Product teams declare intent (image, resources, health checks, warm-pool bounds, trusted hosts); the control plane owns execution. The authors are explicit that this is the Kubernetes operator pattern applied to sandboxes — and it works because the API surface is tiny (`GetOrCreate`, `Get`, `Delete`) while the hard problems (provider failover, egress policy, credential handling) live entirely behind it. The same pattern shows up in [[SPACE — Perplexity's Firecracker Sandbox Platform]] and [[We Built a Scalable Agent Sandbox]]: when AI products need untrusted-code compute, the winning design is a thin declarative API over a thick platform, not a library.

> "The naive solution would have been to implement a strict network allowlist, but that's often too blunt for the products we want to build. AI workloads may need to search and read arbitrary pages on the internet, which an allowlist would prevent."

This is the sharpest idea in the piece. Sandboxes for *agents* face a genuinely different threat model than sandboxes for CI: the workload needs open-ended read access to do its job. Figma's answer — reads anywhere, writes only to named trusted hosts, credentials encrypted at rest and swapped only at the egress boundary — is a policy shape that acknowledges agent workloads need the internet but shouldn't be able to *act* on it. It complicates [[Fences, not Sandboxes]] productively: that argument says containment beats prohibition, and Nimbus is a concrete, at-scale implementation of exactly that stance. It also pairs well with [[Attack Review -- Claude Allowlisted-Egress Exfiltration]], which shows what happens when the egress side of this problem is gotten subtly wrong.

The reliability half is essentially distributed systems under another name: exactly-once sandbox claims via a single Postgres transaction with `FOR UPDATE SKIP LOCKED`-style semantics, demand-based pool sizing instead of fixed targets, provider routing weighted by recent outcomes with careful discrimination between client cancellations, self-imposed rate limits, ambiguous timeouts, and confirmed failures. The control-loop framing — observe pool state, act, repeat — plus a production-conditions simulator is a strong argument that sandbox platforms are control systems and should be built and tested like one. The bug the simulator caught (batch-wait draining warm pools toward a slow-but-"healthy" provider) is a classic monitoring failure: success counted at the wrong granularity.

> "Most product teams will never know which path served their sandbox, which provider created it, or which daemon kept its processes healthy—which is what we intended."

The closing metric of platform success is invisibility. Nimbus moved incident response from paging humans to rerouting automatically — the same trajectory [[Platform Engineering as the AI Control Plane]] argues for generally: as agent infrastructure becomes load-bearing, the platform layer absorbs the operational burden rather than each product team.

**My take.** The read/write egress split is the detail worth stealing; most sandbox write-ups stop at "we isolate the workload," and Nimbus is unusually honest that isolation alone doesn't answer "what may this workload *say*?" The warm-pool and routing material is solid but fairly standard SRE craft dressed in sandbox clothes — the interesting claim is that provider-agnostic policy enforcement (sidecar where the provider can't help) is what makes multi-provider warm pooling safe at all. Worth noting the piece is a Figma product-launch-adjacent blog post (it doubles as the intro to a three-part series and a hiring ad), so the absence of numbers on cost, attacker attempts, or false-positive rates is conspicuous. Still, it's one of the better first-party accounts of sandbox infrastructure for AI agents.

## Key themes

#concept — sandbox-as-contract: declarative namespaces over an opaque control plane
#pattern — read/write-split egress policy and egress-time credential decryption
#tool — Nimbus, `sandboxd`, `sandbox-base`
#concept — warm pools as both latency fix and availability buffer

---

*Sources: [[raw/how-figma-manages-sandboxes-at-scale]], [[summary/how-figma-manages-sandboxes-at-scale]]*
*Last updated: 2026-11-06*
