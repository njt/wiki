# Distributed Systems

This is the thinnest topic in the wiki, and that thinness is itself the finding. Agent orchestration IS distributed systems -- the primitives of coordination, fault tolerance, and state management keep getting reinvented by teams that don't realize they're building distributed systems. [[Process-Based Concurrency BEAM OTP]] documents the pattern of reinvention: Java reinvented it with Akka (2009), Node.js with worker threads (2010s), and AI frameworks with OTP-like patterns (2020s). The BEAM VM solved these problems in 1986. The question isn't whether BEAM is right -- it clearly is -- but whether the AI ecosystem will ever adopt it, or keep reimplementing it in Python.

---

## The Landscape

### The BEAM Model

[[Process-Based Concurrency BEAM OTP]] makes the strongest case: BEAM processes are ~2KB each (millions per machine), have their own heap and GC (no system-wide pauses), are preemptively scheduled (~4,000 reductions per timeslice), and completely isolated (memory access between processes is impossible, not just discouraged). Hot code swapping lets you deploy without disconnecting active sessions.

"Let it crash" separates business logic from error recovery. Supervisors manage failure at a higher level with strategies (`:one_for_one`, `:one_for_all`, `:rest_for_one`). The mapping to AI agents is direct: agents are long-lived, stateful, failure-prone, and concurrent -- exactly the problem BEAM was built to solve.

[[Loomkin]] is the proof of concept: a multi-agent platform on Elixir/OTP where each agent is a GenServer under a DynamicSupervisor. Spawn in under 500ms, communicate via PubSub in microseconds, support 100+ concurrent agents per node. The decision graph (persistent DAG of goals and decisions), context mesh (offload to Keeper processes with staleness tracking), and self-healing teams (OTP supervision trees) are the most ambitious agent architecture in the wiki. 77K LOC, 2,700+ tests, 328 source files.

### Consensus and Replication

[[celld]] (Deno's self-hosted Durable Objects) is the most extreme brokerless design in the wiki: nodes coordinate entirely through S3 Compare-And-Swap with no consensus protocol, no leader election, no gossip. Each cell is a SQLite database replicated to S3 via a vendored Litestream port; ownership is a conditional PUT. The trade is simplicity of coordination for latency on every cold start. The celld.dev pitch frames the same design as an operational-sovereignty argument rather than a consensus one: because ownership is a CAS lease in *your* bucket and each cell's SQLite and LTX files sit on *your* disk, "what happened to my cell" is answered with sqlite3 and grep, not a vendor status page.

[[Graft]] provides the database layer: SQLite replicated via object storage without a running cluster. Stateless on S3 is architecturally simpler than Raft consensus (rqlite) or FoundationDB (mvSQLite). The conflict resolution is pushed to the application, which is honest about a hard problem that CRDTs ([[Graft]]'s comparison with cr-sqlite) try to solve automatically.

[[Write Snapshot Isolation]] addresses transactional correctness: standard snapshot isolation checks for stale writes when it should check for stale reads. One conceptual fix achieves serializability. The practical lesson: correctness should be structural, built into the isolation mechanism, not bolted on as detection.

### Agent Orchestration as Distributed Systems

[[Scaling Long-Running Agents]] is Cursor's finding that flat self-coordination fails: agents hold locks forever, avoid hard work, slow to a crawl. The planner/worker/judge hierarchy solves it -- but this is just supervisor trees by another name. The pathologies they describe (lock contention, risk aversion, diffusion of responsibility) are textbook distributed systems failure modes.

[[klaw.sh]] applies Kubernetes concepts to agents: lifecycle management, namespace isolation, cron scheduling, distributed controller + worker deployment. The kubectl metaphor is apt because the operational needs are identical.

[[Zeroclaw]] (31.3k stars, Rust) handles 30+ communication channels with trait-based interfaces, OS-level sandboxing, and cryptographic tool receipts. The distributed aspects: multi-node potential, fallback chains across providers, and the agent coordination primitives that every distributed system needs.

## Key Tensions

**BEAM's correctness vs. Python's ecosystem.** BEAM is the right architecture for agent orchestration. Python dominates the AI ecosystem. "Rewrite it in Elixir" is not actionable advice for most teams. The pragmatic answer: build your Python/Go/Rust agent framework with BEAM's principles (isolated state, message passing, supervision) even if you can't use BEAM's runtime. But approximating in userspace what BEAM provides at the VM level is always a pale imitation.

**Centralized vs. decentralized coordination.** [[Scaling Long-Running Agents]] uses hierarchical decomposition with a central planner. [[Cord]] uses dynamic task trees where agents discover coordination structures at runtime. [[workgraph]] keeps the graph centralized but lets agents come and go. The traditional distributed systems debate (centralized coordinator vs. peer consensus) replays in agent orchestration.

**Synchronous vs. asynchronous coordination.** [[Zero Alignment]] argues that existing tools are single-player, asynchronous interfaces. Agent orchestration needs real-time coordination. But synchronous coordination doesn't scale to 100+ agents. The resolution is probably the same as in distributed systems: synchronous within a team, asynchronous between teams, with explicit handoff protocols.

## What's Missing

This section needs to be honest about the gaps, because they're large.

**Consensus protocols for agents.** How do multiple agents agree on a shared state? Raft, Paxos, and PBFT solve this for databases. Agent orchestration systems mostly ignore it, using single-writer patterns (SQLite, JSONL files) or accepting eventual consistency without formalizing it. As agent systems scale to dozens or hundreds of concurrent agents, consensus becomes unavoidable. [[Aurora DSQL]]'s distributed adjudicators and Journal replication show one production approach to the problem, though at database scale rather than agent scale. [[State-Oriented Consistency]] offers a complementary insight: the hard problem isn't *how* to achieve consensus — it's identifying *which* state actually needs it, and preventing consensus from colonizing the rows that don't. The Keel IoT team's five-row classification table (live sessions → coordinator, offline sessions → deterministic placement, routing → available, durable messages → durable storage, membership → gossip) is a worked example of asking each piece of state what it actually requires rather than defaulting to one answer for everything.

**Partition tolerance.** What happens when an agent loses connectivity to the coordinator? To other agents? To external services? The CAP theorem applies to agent systems as surely as to databases, but nobody is designing for it explicitly. [[Loomkin]]'s OTP supervision trees handle process failures but not network partitions.

**Distributed transactions.** An agent that needs to update a database AND send an email AND commit code is executing a distributed transaction. [[Designing a Passively Safe API]]'s outbox/inbox pattern solves this for HTTP APIs. Nobody has generalized it for agent tool-use workflows.

**Observability at scale.** [[AgentsView]] provides analytics for 24+ coding agents. But distributed systems observability (distributed tracing, causality tracking, latency histograms) is absent from the agent tooling space. When 50 agents are working on related tasks, understanding what happened requires the same tools (Jaeger, OpenTelemetry) that microservice architectures use.

**Formal verification.** Distributed systems use TLA+ and similar tools to verify protocol correctness before implementation. Agent orchestration protocols are specified informally (if at all). As these systems take on higher-stakes work, formal verification of coordination protocols will become necessary.

**More BEAM/OTP case studies.** [[Loomkin]] is the only BEAM-based agent system in the wiki. Given the argument that BEAM's model is correct for this problem, the wiki should track more Elixir/Erlang agent projects as they emerge.

## Key Themes

#distributed-systems #BEAM #consensus #coordination #fault-tolerance #agent-orchestration
