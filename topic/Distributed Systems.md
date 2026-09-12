# Distributed Systems

Distributed systems is the discipline of building software that keeps working when its parts do not — and the wiki's conclusion is that the hard questions are not new, but their answers have sharpened into one principle: correctness is not a property of the transport or the consensus protocol, but of decisions made per piece of state and per source, and enforced at the write boundary. The eight fallacies of distributed computing remain true decades after they were collected at Sun, and every mechanism examined here — webhook, poll, bus, queue, sync engine — is a different way of coping with the fact that delivery is at-least-once, bandwidth is finite, and no single administrator has the whole picture. Systems that design for that reality stay up; systems that treat it as a bug to be fixed quietly fail.

---

## The Argument

### The eight fallacies are the vocabulary

The fallacies were assembled at Sun Microsystems — the first four by Bill Joy and Tom Lyon, three more by L. Peter Deutsch, and an eighth by James Gosling — and aimed squarely at people writing network software. Two decades later George Michaelson finds each one alive and well: home Wi-Fi is a bottleneck (a phone that can sustain 400 Mbit/s talking to a five-year-old router capped at 100 Mbit/s), BGP topology changes break the assumption of a static network, traffic analysis defeats even encrypted connections, and IP masks the difference between local and remote, fast and slow. The Internet is "probably broken somewhere, for some users, at all times." The list is the vocabulary for everything else on this page: each failure mode that follows is a specific way of believing one of these eight things. Even the meta-layer is unreliable — Michaelson notes the list's own history is routinely garbled when people call Tom Lyon "Dave Lyon."

### Delivery is at-least-once, and correctness lives at the write boundary

Michel Tricot's survey of trigger architectures makes the case that the choice between event-driven and polling is a false binary, and that "real-time webhooks" are mostly marketing. Across Stripe, Shopify, HubSpot, and GitHub the underlying contract is the same: at-least-once, unordered, best-effort. Stripe retries for up to three days and explicitly does not guarantee order; Shopify retries eight times over four hours and, worse, its retries carry the *original* payload rather than freshly fetched data; HubSpot retries ten times over a day; GitHub is the outlier that barely retries at all, forcing consumers to poll its deliveries API or run a parallel sweep. The naive alternative, polling, hits rate-limit ceilings faster than teams expect: polling 100 GitHub endpoints every 30 seconds is 2.4× the 5,000-request hourly cap, and HubSpot's per-app burst budget "degrades linearly with scale" as tenants are added.

The resolution is not to pick one side but to accept that "the source system has already made much of the decision." A source with no webhooks (Odoo, Sage Accounting, Pennylane) is a polling source; a source with a durable bus is a bus source. Salesforce alone offers three overlapping event-driven mechanisms — Outbound Messages, Platform Events, and Change Data Capture — whose semantics genuinely differ, and CDC's replay IDs are positional markers that stop working when Salesforce moves an org to a new instance. The thread running through every mechanism is that none of them is exactly-once: "No trigger is exactly-once on its own. Exactly-once effects come from idempotent handlers at the write boundary." The honest default for an HTTP-only source is therefore a hybrid — a webhook fast path plus a reconciliation poll that backfills whatever the webhook missed, because "events will be missed. Not might. Will."

Agents make idempotency harder, because an LLM is a non-deterministic client: a retry that regenerates slightly different tool parameters hashes to a different key and becomes a duplicate write. The fix is to key on stable structural context — `(agent_run_id, step_id, tool_name, call_index)` — assigned before inference and unchanged on retry. The final requirement is that the trigger land somewhere that can hold state across a fifteen-minute call or a human-approval wait; a webhook landing in a stateless function is fragile by design. This is where Temporal's workflows-and-signals and Inngest's memoised steps enter — durable execution is what keeps a retry from re-burning tokens on work already done.

### Overload is a different failure than delivery

Fred Hebert's "Queues Don't Fix Overload" is the corrective to message-bus enthusiasm. A queue is a buffer, and a buffer only absorbs *temporary* overload; under prolonged overload it fills and the failure becomes more catastrophic, not less — "you're making failures more rare, but you're making their magnitude worse." The real work is finding the bottleneck — the "red arrow" — and then doing one of two honest things: back-pressure (refusing input) or load-shedding (dropping it). The natural slowness of an overloaded system was already back-pressure; adding a queue to "fix" it removes the safety mechanism. Hebert ties this to the end-to-end principle: persistent queues turn calls into fire-and-forget, severing the channel by which errors reach the caller. His prescription is idempotent APIs, so callers can "safely retry requests and know if they worked" — the same idempotency Tricot arrives at from the trigger side. The two meet at the write boundary: one says make the handler idempotent, the other says make the API idempotent.

### Async work breaks the transport

[[All Your Agents Are Going Async]] takes the delivery problem up a level. When agent work moves from a chat window to cron jobs, webhooks, and phone-driven sessions, "the lifetime of an agent's work is decoupled from the lifetime of a single HTTP connection" — and HTTP, built around request-response, cannot follow. Four scenarios break it: the agent outlives its caller (a cron finishes with no one listening), the agent wants to push unprompted, the caller switches devices mid-task, and multiple humans share one session. Zak Knill splits the problem into durable state (where the session lives) and durable transport (how bytes travel across disconnects), and argues that Anthropic's Routines and Cloudflare's Sessions API solve only the state half — "half works, but it's not 'art of the possible'." [[Cloud Agent Lessons from Cursor]] lands on the same decomposition independently: Cursor decouples agents, machines, and conversation state into three components, runs the agent loop in Temporal, and streams conversation state to clients through an append-only storage layer that tolerates retries. Both are describing the same move — from a connection to a durable thing you can leave and rejoin.

### Consistency is decided per state, not per system

[[State-Oriented Consistency]] generalises the lesson inward. The Keel IoT team's clustered MQTT broker died of an OOM kill whose root cause was not a memory leak but an unexamined assumption they name Uniform Consistency: every pod loaded every client's session state because the architecture assumed every node had to be able to serve any client. The correction is a five-step loop — identify the state, ask what happens if two nodes disagree, find the minimum guarantee, choose the weakest mechanism that provides it, repeat — and a table mapping live sessions to a coordinator, offline sessions to deterministic placement, routing to an available (AP) strategy, durable messages to durable shared storage, and membership to gossip. The striking conclusion is that consensus is not the hard part: "consensus was recognized as a solved problem; the hard decision was identifying which parts deserved it, and refusing to let it creep into rows that didn't need it."

### Replication generalises only so far

The most systematic treatment is Mikael Siidorow's thesis on the limits of generalising client-side sync. Across eight architectural dimensions it finds that offline capability is the most discriminating axis; that the field clusters into database-pluggable engines (Zero, ElectricSQL, PowerSync), bundled platforms, CRDT-decentralised libraries, and a long tail of singular designs; and that five limits resist generalisation. Authorization must stay separate from the sync mechanism; global invariants like uniqueness require consensus, pushing practical scale down to roughly 50,000 objects; the write path refuses to generalise (Notion could not use standard CRDTs, Figma maintains two sync engines); browser offline storage is still immature; and partial sync remains unsolved. Last-write-wins dominates production even though conflict resolution draws the academic attention.

Stash, Telepath Computer's folder-sync CLI, is a worked example of those trade-offs. It diffs folders against a flat JSON snapshot of `{path: hash}` rather than a commit graph, merges concurrent text edits with Google's diff-match-patch (three-way, base versus local versus remote), and falls back to last-modified-wins for binaries. Its honest gambles are the ones the thesis predicts: it accepts last-write-wins for structured files, and it loses local history in exchange for a simple provider contract. The sharpest observation is that the innovation is the snapshot-based diffing itself — a flat hash map that avoids CRDTs, operational transforms, and version vectors entirely — and the sharpest risk is that text-merging a JSON config edited from two machines at once is a "footgun waiting to happen."

### The operational floor: environments, failure modes, and the network plane

The remaining sources fill in the operational ground. [[Cloud Agent Lessons from Cursor]] argues that "the development environment IS the product": a laptop agent inherits a shell, tools, keys, and repo state, and reconstructing all of that in the cloud is the hard part, because an incomplete environment degrades output quality subtly rather than erroring loudly. Cursor's numbers are the clearest durable-execution evidence in the set — a work-stealing architecture that ran at roughly one nine of reliability, and a migration to Temporal that took it past two nines. [[SDPD — Systems Design Police Department]] compresses the whole field into a game of 33 named failure modes — split brain, Byzantine witness, thundering herd, cache avalanche, retry storm, DNS disaster, saga failure — which doubles as a checklist of everything a "distributed system" can mean in practice. And [[Headscale]] illustrates the networking plane: a self-hosted reimplementation of the Tailscale control server, it shows how thin the coordination layer actually is — key exchange, IP assignment, ACLs, DNS, relay — while the data plane rides WireGuard with NAT traversal. It is a concrete answer to the sixth fallacy, that there is one administrator.

## Where the Sources Disagree

**Queues: band-aid or foundation.** Hebert argues that queues do not fix overload and that persistent queues damage the end-to-end principle; Tricot argues that when a source publishes a durable, replayable bus, you should prefer it, because the bus's replay replaces a parallel reconciliation poll. These are compatible once you notice they answer different questions — Hebert is attacking the queue-as-buffer for overload, Tricot is using the bus for durable delivery and replay, not for absorbing excess. But the emphasis genuinely differs: Hebert would see "put Kafka in front of it" as the doomed optimization cycle he describes, while Tricot treats the bus as the best available answer for HTTP-only sources. Hebert is the more convincing general warning; the arbitration is his own point — you have to find the red arrow first, because whether a bus helps depends entirely on what the bottleneck is.

**HTTP: patch it or replace it.** Tricot's whole framework is built on webhooks — HTTP POST — plus a reconciliation loop, an elaborate patch to make HTTP tolerable. Knill argues HTTP is simply the wrong transport for async agents and wants a durable realtime session instead. Both agree on the underlying reality (at-least-once delivery, no durable connection); they disagree on whether to layer reconciliation over HTTP or abandon it. Cursor splits the difference, using streaming delivery but Temporal for durability. The most defensible reading is that these are answers at different layers — Knill's durable transport is effectively a bus, and Tricot's reconciliation is what you do when the bus is not yours to build.

**Conflict resolution: last-write-wins in practice, CRDTs in theory.** Siidorow finds last-write-wins dominating production while CRDTs draw the academic attention; Stash treats text-aware three-way merge as its reason to exist, against Syncthing's and Dropbox's uniform last-write-wins. The tension is whether merge quality is worth the complexity. Siidorow's production evidence is the stronger guide: the write path is too application-specific for a general answer, which undercuts the CRDT-decentralised cluster's claim to generality — Stash's merge works precisely because it scopes itself to text files.

## What's Missing

**Consensus mechanics.** The sources assert that consensus is a solved problem and that uniqueness requires it, but none actually walks through a consensus protocol; a reader wanting to know *how* Raft or Paxos works will not find it here.

**Partition tolerance.** The fallacies name topology change and SDPD names split brain, but no source designs for a network partition explicitly — the CAP theorem is never invoked by name.

**Observability and forensics.** Cursor reports reliability nines without the instrumentation story behind them. Distributed tracing, causality tracking, and latency histograms — the tools for seeing a system misbehave — are absent from the evidence.

**Formal verification.** Nothing in the set mentions TLA+ or equivalent, even though the delivery contracts described are exactly the kind of protocol that formal methods exist to check.

**Cost.** Fallacy seven names transport cost, but no source quantifies what any of these patterns costs to operate; the hybrid pattern's "doubles operational cost" is asserted, not measured.

## Also on This Theme

- [[Process-Based Concurrency BEAM OTP]] — the BEAM VM's concurrency and fault-tolerance model: lightweight isolated processes, supervision trees, "let it crash."
- [[Loomkin]] — a multi-agent platform on Elixir/OTP that treats each agent as a supervised GenServer.
- [[ZeroMQ — Universal Messaging Library]] — message-passing primitives (pub-sub, push-pull, request-reply) as an embeddable library.
- [[celld]] — Deno's brokerless Durable Objects, coordinating through S3 compare-and-swap rather than a consensus protocol.
- [[Graft]] — SQLite replicated via object storage without a running cluster, with conflict resolution pushed to the application.
- [[Write Snapshot Isolation]] — the argument that snapshot isolation should check for stale reads, not only stale writes.
- [[Scaling Long-Running Agents]] — Cursor's finding that flat agent self-coordination fails, and a planner/worker/judge hierarchy as the fix.
- [[klaw.sh]] — Kubernetes concepts (lifecycle, namespaces, cron, controllers) applied to agents.
- [[Zeroclaw]] — a Rust agent framework spanning many communication channels, with sandboxing and receipts.
- [[Cord]] — dynamic task trees where agents discover coordination structures at runtime.
- [[workgraph]] — a centralised task graph that agents join and leave.
- [[Zero Alignment]] — the argument that existing agent tools are single-player and asynchronous, where coordination needs real-time structure.
- [[Aurora DSQL]] — distributed SQL with adjudicators and journal replication, as one production answer to consensus.
- [[Designing a Passively Safe API]] — the outbox/inbox pattern for reliable HTTP APIs.
- [[AgentsView]] — analytics across many coding agents.
- [[Aspire]] — the .NET observability reference point that injects OpenTelemetry by default.
- [[Introducing Claude Tag]] — companion note to the async-agents source.

---

*Compiled from 10 sources: [[summary/21-years-eight-fallacies-distributed-computing]], [[summary/all-your-agents-are-going-async]], [[summary/cloud-agent-lessons]], [[summary/event-driven-vs-polling-architectures]], [[summary/headscale]], [[summary/queues-dont-fix-overload]], [[summary/sdpd-live]], [[summary/stash-telepath-computer]], [[summary/state-oriented-consistency-html]], [[summary/the-limits-of-generalized-sync]]*
*Last compiled: 2026-09-12*
