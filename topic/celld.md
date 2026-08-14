# celld

Self-hosted, distributed Durable Objects from Deno: an open-source daemon that
runs Cloudflare Workers and Durable Objects on your own machines, using
object-storage Compare-And-Swap as the sole coordination mechanism. Each cell
(Durable Object) is its own SQLite database replicated to S3; the output gate
withholds every write response until the replicator proves it durable, so the
system never acknowledges a write it could lose. The cleanest example I've seen
of a brokerless distributed system built on a state-machine architecture where
the decision core is provably separable from I/O.

---

## The Sovereignty Argument (celld.dev)

The marketing site makes an argument the README only implies: celld's value isn't
just architectural elegance, it's **operational sovereignty** — keeping the
Durable Objects programming model while reclaiming placement, state, and
evidence:

> "Durable Objects is a strong programming model. celld keeps that model while
> moving placement, state, and operational evidence into infrastructure you
> choose."

Three beats, each aimed at a different Cloudflare pain point.

**The bucket as coordinator, restated plainly.** The README describes S3 CAS as
an implementation detail; the site makes it the headline:

> "The bucket is the coordinator — no membership protocol, no failure detector,
> no consensus. Ownership is a record in your bucket, claimed with one atomic
> write. celld's built-in replicator continuously ships each cell's SQLite state
> to that bucket as LTX segments."

This is the same mechanism as the `ReadingOwner` → `Acquiring` phase machine,
compressed into a sentence a CTO can repeat. The "one atomic write" is the
conditional PUT.

**Tenancy isolation, not just failover.** The second beat reframes the lease:

> "A cell's identity isn't fused to a machine — ownership is a lease in your
> bucket, granted by compare-and-swap. Lose a node and another acquires the
> lease and restores the cell in seconds: your fleet reading your storage, not a
> vendor restoring a placement you can't see."

The load-bearing word is "tenancy." What you're buying isn't merely faster
failover — it's that no shared scheduler or placement layer can couple your
workload to another customer's:

> "Your fleet still depends on its machines, network, and bucket provider. What
> changes is tenancy: no shared Durable Objects scheduler or placement layer can
> couple your application to another customer's workload."

This is the rare self-hosted pitch that declines to oversell. It concedes the
failure surface doesn't shrink — it *moves*, from "Cloudflare is down" to "my
machines, network, and bucket provider are down." What you buy is isolation and
visibility, not reliability.

**Operational evidence, the sharpest claim.** The third beat is the one that
most cleanly separates celld from a managed service:

> "When a cell misbehaves the evidence is on your disk — the ownership record,
> the SQLite and LTX files, and the logs. You answer 'what happened to my cell'
> with sqlite3 and grep, not a status page that declines to say."

The engineering bet is that `sqlite3` and `grep` on files you own beat a managed
dashboard for debugging, precisely because the answer is provable rather than
asserted — the same ethos as [[Learning a Few Things About Running SQLite]],
which treats operations as something you learn by *doing*, against your own data.

Critically, this is marketing copy: pithy where the README is precise, and silent
about the hard parts the README documents (per-thread V8 isolates, HMAC-only peer
auth with no TLS, the latency cost of CAS on every cold start). Read the site for
the *why* and [[raw/celld]] for the *how*.

## Architecture

celld is a Rust workspace (`crates/celld`, `crates/logic`, `crates/ltx`) with a
single binary. The architecture divides cleanly into three layers with
hard-enforced boundaries:

**The Decision Core (`crates/logic/lib.rs`)** is a pure, deterministic state
machine with zero dependencies — no async, no I/O, no clocks, no randomness, no
locks. The only way behavioral state advances is through the single function
`on_event(&mut State, Event) -> Vec<Effect>`. The `State` struct (~1,050 lines)
holds every piece of coordination authority: per-cell phases (an 18-variant
`Phase` enum tracking cells from `Dormant` through `ReadingOwner`,
`Acquiring`, `Restoring`, `Resident`, `Fenced`, etc.), node lease state,
activation permits, hibernation permits, capacity waiters, shed floors, and
gated write tracking. This is what makes the system deterministically replayable
— the same event sequence always produces the same effects and state.

**The Executor (`crates/celld/main.rs`)** is a single serial actor (~5,400
lines) that owns the core `State`, feeds it events from an mpsc channel, and
performs the returned `Effect`s. The actor's `run()` method sits in a
`tokio::select!` loop polling messages, completed effect futures, and timers
from a `DelayQueue`. Every I/O effect (S3 read/write, V8 isolate start/stop,
alarm dispatch) is spawned as a boxed future; completion returns a versioned
event through the mailbox. The actor never blocks.

**The Replicator (`crates/ltx/`)** is a vendored Rust port of Litestream v0.5
(via rustyriver), repurposed as an in-process library for SQLite WAL replication
to S3 in the LTX format. It serves celld's RPO=0 durability guarantee: the
`await_durable` primitive proves a specific WAL position is on the bucket before
the output gate releases a write response.

**The Runtime (`crates/celld/runtime.rs`)** manages V8 isolates. Each cell gets
its own OS thread with its own V8 isolate, spawned via `std::thread::Builder`.
Stateless Worker requests go to a shared thread pool. The `RuntimeManager` owns
a `CellRegistry` split into `starting` and `published` handles — a cell becomes
routable only after explicit publication, not merely after isolate startup.

### How cells route

A Durable Object request follows a deterministic path through the phase machine:

1. **Dormant** → request arrives → queue for activation permit (bounded by
   `max_activations`)
2. **ReadingOwner** → S3 read of `cells/<name>.json` to learn who owns it
3. **ReadingNodeLease** → if owned, check the owner's node lease is live
4. **Acquiring** → S3 Compare-And-Swap to claim ownership (CAS guard based on
   prior etag or absent-key)
5. **Restoring** → pull SQLite snapshot from bucket or use local cache
6. **Starting** → spawn V8 isolate thread, load Worker, run Durable Object
   constructor
7. **Publishing** → register the cell in the runtime's published map, making it
   dispatchable
8. **Resident** → serving requests

At each stage, an `OperationDeadline` timer fires if the effect doesn't complete
in time, preventing swallowed effects from parking requests forever. A deadline
on a read is `Failure::Definite` (safe to retry); on a write is
`Failure::Ambiguous` (must reconcile, not retry).

### Node leases and fencing

Node authority comes from a lease object in the bucket (`nodes/<id>.json`),
renewed every TTL/3. Three modes: `Continuous` (always renew), `Lazy` (only
acquire when local work exists), and `Shadow` (renew but report when idle long
enough to safely release). A self-fence timer fires at TTL+1ms — if the lease
hasn't been confirmed by then, the node halts with exit code explaining why.
This is the safety invariant: a node that cannot prove it owns what it's serving
must not serve.

### The output gate (RPO=0)

When a write request completes, its response is NOT sent to the client. Instead,
the executor calls `gate_write()` which sends a `Message::GateWrite` to the
actor. The core emits `Effect::AwaitDurable` with the write's WAL position. The
replicator proves that position durable on the bucket, then the core emits
`Effect::ReleaseResponse`. Only then does the HTTP response go out. This is the
same per-object output gate Cloudflare provides — celld reimplements it. A
failed proof breaks the response (the client sees an error, not a false ack).

WebSocket messages have their own per-cell output gate: frames are queued in
write-order behind barriers, flushing only as each barrier's durability proves.
A single failed barrier breaks the entire cell gate and resets all sockets.

## Key Techniques

**Object-storage CAS as the only coordination primitive.** There is no Raft, no
Paxos, no gossip protocol, no failure detector. Nodes discover each other by
listing `nodes/` in the bucket; they fence each other by writing ownership
records with S3 conditional PUTs (If-Match / If-None-Match). The bucket IS the
control plane. This works because the consistency model needed — single-writer
per cell — maps exactly to what S3 conditional writes provide. The bucket client
(`crates/celld/bucket.rs`) uses two separate `AmazonS3` instances: one with
retries for ordinary reads/writes, and one with `max_retries: 0` for CAS
operations, because a retried CAS that lands on the first attempt's own ETag
change would report a false rejection.

**Deterministic, replayable decision core.** `celld-logic` has `[dependencies]`
that is literally empty in its `Cargo.toml`. This is not an aspiration — it's a
hard constraint. The README says "The runtime and compatibility surface are
still evolving… a deterministic simulation of the distributed protocol under
fault injection, run before each release." The simulation is possible because
the core is a pure function from (State, Event) to (State, [Effect]). There are
async variants (`RequestAt`, `WakeHintAt`, `CapacityRequestAt`) that carry
wall-clock and monotonic timestamps inside the event, so the core never reads a
clock itself. `State::validate()` runs after every event in debug builds,
checking invariants like occupancy ≤ max_resident, no double-queued waiters, no
gated writes on unpinned requests, and that every active request has a Resident
cell.

**Phase-specific failure handling.** Ambiguity is classified per-effect, not
globally. A failed S3 read is `Failure::Definite` (nothing committed). A failed
CAS is `Failure::Ambiguous` (may have committed). This classification determines
whether the core retries, reconciles, or fails the request. An ambiguous acquire
triggers a re-read of the owner record (not a blind retry), up to
`MAX_ACQUIRE_RECONCILES` (3) times before failing.

**Pressure shedding as deterministic policy.** The `LoadSampled` event carries
RSS, CPU, and resident-cell count into the core. The core decides whether to
latch shedding, computes a `shed_floor`, and triggers LRU evictions one at a
time (bounded by `max_hibernations` concurrency). Eviction order uses
`last_used_mono_ms` and a `hibernation_refused_mono_ms` backoff to avoid
repeatedly picking the same failing cell. The latch has hysteresis: it turns on
at the high watermark and off at the low. The executor re-publishes load numbers
to `ownership_index_generation` for peer capacity ranking.

**Capacity handoff with epoch-zero candidates.** When a node cannot serve a
cell, it selects a peer from the capacity index (recent node leases with load
samples) and forwards the request as a `CapacityRequestAt`. The receiving node
checks whether its advertised capacity is still accurate; if not, it refuses
immediately so the sender can try another candidate. An epoch of zero means
"this is a candidate, not an owner" — a refusal disproves one load sample
without consuming the stale-route budget.

**Lazy node leases for idle fleets.** In Lazy mode, a node with no local cells
releases its lease entirely and goes dormant. The next request first acquires
the lease, then proceeds with cell ownership. This means a fleet of idle nodes
generates zero S3 traffic for lease renewal. `CELLD_LEASE_LINGER_MS` controls
how long a node waits before releasing after its last cell departs.

**Alarm system integrated into the core.** Durable Object alarms are wall-clock
timers stored in the cell's own SQLite database. The core tracks alarm state
(`Armed` / `Firing`), schedules monotonic timers, and upon firing dispatches
`Effect::FireAlarm` which runs `alarm()` in the V8 isolate. A separate
`WakeFlusher` system maintains bucket entries for pending alarms so that other
nodes can discover cells that need attention even when the owning node is down.

## Design Decisions

**Split the decision core from the executor, at the cost of indirection.**
Every I/O effect goes through the effect enum → boxed future → completion event
path, adding latency and complexity compared to calling S3 directly. But the
payoff is determinism: the entire system can be replayed from an event log, and
the simulation can inject faults at every effect boundary. The Cargo.toml
comment captures this: "One actor serializes every event through celld-logic;
the actor polls its mailbox, timers, and in-flight effect futures together. This
is the execution shape required for monotonic lease ticks to fence the node even
when a storage operation remains hung."

**S3 as the sole coordination fabric, sacrificing latency for simplicity.**
Every ownership change is a conditional PUT. Every durability proof is an S3
HEAD + object listing. The round-trip cost is paid on every cold start and every
eviction. But the alternative — a consensus protocol with membership, leader
election, log replication — is a distributed systems problem with well-known
failure modes. S3 conditional writes are a simpler primitive with a simpler
failure model: either the write applied or it didn't, and if you can't tell, you
re-read. The `put_cas` contract in `bucket.rs` is explicit: `Ok(None)` only for
clean 412/409 rejection; everything else is an error (ambiguous).

**V8-per-cell thread model, not an async runtime.** Each cell is an OS thread
with its own V8 isolate. This is expensive in memory (a V8 isolate is ~5-10MB)
but means a crashing or looping cell cannot affect others. The core enforces
concurrency limits (`max_resident`, `max_activations`) to bound this. Stateless
Worker requests use a shared thread pool because they don't carry state and are
inherently isolated by V8's context separation.

**No TLS on the peer protocol, by design.** The README and security docs are
explicit: peer HTTP is plain. Every node-to-node request is HMAC-authenticated,
body-signed, clock-bounded, and replay-protected, but not encrypted. The design
assumes a trusted private network or WireGuard/Tailscale overlay. This avoids
the certificate management problem at the cost of requiring operators to set up
their own encryption layer.

**Pull requests disabled.** The README states this plainly: "Coding agents make
it too easy to send a large, low-context change that costs maintainers more time
than it saves." Contributions arrive as `git format-patch` emails. This is an
explicit stance on contribution quality over volume, made more interesting by
the fact that the author (ry@deno.com — Ryan Dahl, Deno's creator) has both the
standing and the leverage to enforce it.

## Comparison Notes

**vs. Cloudflare Durable Objects**: This is a self-hosted reimplementation of
the same abstraction — per-object SQLite, RPO=0 output gate, alarm API, Worker
entry point. The key difference is the coordination fabric: Cloudflare uses
their internal control plane; celld uses S3 CAS. Cloudflare's DOs run in
Cloudflare's V8 isolate infrastructure; celld runs them in per-thread isolates
on your own machines. The `docs/cloudflare-compat.md` page maps the
compatibility surface explicitly.

**vs. Fly.io Machines with LiteFS**: Both replicate SQLite to S3 and both
support single-writer-per-database. Fly.io uses a Consul-based lease system;
celld uses S3 CAS directly. LiteFS is a separate process; celld's `celld-ltx`
runs in-process. Fly.io provides the infrastructure (proxy, certs, placement);
celld provides none — you bring your own network, TLS termination, and load
balancing.

**vs. Turso / libsql**: Both distribute SQLite. Turso uses a primary-replica
model with a central control plane; celld is fully peer-to-peer with no central
service. Turso is a database product; celld is an application runtime where the
database is an implementation detail of the Durable Object abstraction.

**vs. State Machine Replication (Raft/Paxos)**: celld deliberately avoids
consensus. It can do this because the Durable Object model guarantees
single-writer access: if two nodes ever run the same cell, the epoch fence
prevents the stale one from committing writes. The cost is that failover is not
instantaneous — a new owner must notice the old lease expired, CAS the ownership
record, restore from S3, and start the isolate. This is seconds, not
milliseconds.

**vs. [[Apache Burr]]**: Both use explicit state machines as the core
abstraction. Burr is a Python framework for building AI agents as state
machines; celld uses the same pattern but for distributed infrastructure
coordination. The common insight: a state machine makes behavior replayable,
testable, and auditable in a way that ad-hoc async code never can.

**vs. [[The GUS Stack — Go, Unix, SQLite]]**: celld is a Rust counterpoint to
the GUS philosophy. It uses SQLite pervasively (one database per cell), but
instead of Go's simplicity it embraces Rust's type system for correctness (the
`Failure::Definite` vs `Failure::Ambiguous` distinction is enforced at the type
level). The `Cargo.toml` workspace deliberately pins every dependency to a
single version to prevent drift — the same "boring infrastructure" discipline.

---

*Tags: #tool #project #distributed-systems #database #sqlite #edge-computing #state-machine*

---

*Sources: [[raw/celld]], [[summary/celld]], [[raw/celld-dev]], [[summary/celld-dev]]*
*Last updated: 2026-08-14*
