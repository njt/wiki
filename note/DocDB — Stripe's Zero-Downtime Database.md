# DocDB — Stripe's Zero-Downtime Database

Stripe's internal Database-as-a-Service, built atop a custom MongoDB fork, powers $1.4T in payments across 2,000+ shards at 5M QPS. The real story isn't the database — it's the zero-downtime data movement platform that makes horizontal resharding, version upgrades, and tenancy migrations all the same operation. This is database infrastructure where migration is a first-class primitive, not a crisis.

---

## Key Quotes

> "Building DocDB allowed us to bake in security right from the start, specifically through robust enforcement of authorization policies right at the data layer."

Morzaria's build-vs-buy argument is unusually honest. Most companies build because they want to; Stripe built because they had specific security, reliability, and scale requirements that off-the-shelf MongoDB couldn't meet. The authorization-at-the-data-layer point is subtle — it means every query is policy-checked at the proxy, not relying on application-level enforcement that someone will forget.

> "the critical phase of the migration, which is shorter than the duration of a planned database primary failover"

This is the constraint that shaped the entire design. Stripe already tolerates primary failovers — they happen, they're planned for, applications have retry budgets. So the migration cutover just needs to fit inside that same window. Elegant: don't make migrations perfect, make them no worse than something you already handle.

> Sorting data by most common index attributes improved write throughput **10x**

The kind of optimization that only comes from deep understanding of the storage engine. MongoDB's B-tree indexes mean sorted writes hit sequential pages instead of random ones. Obvious in retrospect, but only if you're thinking at the B-tree level during bulk import.

> "We've updated our entire fleet of more than 2,000 database shards using the data movement platform."

Version upgrades via the same mechanism as resharding. This is the platform investment paying compound interest — build one data movement capability, get horizontal scaling, version upgrades, tenancy migration, and split/merge all from the same machinery.

> "The MongoDB oplog itself gives us the idempotency"

No distributed consensus protocol, no two-phase commit for replication — just replay the oplog, which is already designed to be idempotent. Leaning on the database's existing guarantees rather than building new ones. Compare this to the complexity of [[Postgres CDC in ClickHouse, A Year in Review]], where idempotency in CDC is a major engineering challenge.

---

## Key Themes

- **#tool** — DocDB, MongoDB (custom fork), custom proxy layer, routing metadata service, CDC/oplog replication
- **#concept** — **Zero-downtime data movement as platform primitive:** migrations, upgrades, and resharding are the same operation. Build once, use everywhere. **Version gating for traffic switching:** fence the source, bump the version, reject stale requests. Cleaner than distributed locking. **Bidirectional replication for fast rollback:** the target is always ready to become the source.
- **#pattern** — **Infrastructure-as-platform:** proxy + control plane + metadata service + CDC = a database platform, not just a database. Same pattern as [[PgDog]] but for MongoDB and at Stripe's scale. **Build for your existing failure budget:** don't make migrations perfect, make them no worse than failovers you already tolerate.
- **#person** — Jimmy Morzaria (Staff Engineer, Stripe; ex-AWS QLDB/MSK)

---

## Critical Analysis

**The platform investment is the real story, not the database.** Stripe didn't build a better MongoDB. They built a better way to move data between MongoDB instances. The fact that version upgrades, resharding, and tenancy migration are all implemented as the same chunk migration operation is the kind of generality that only comes from good platform design. Most companies build a resharding tool, then a separate upgrade tool, then a separate migration tool. Stripe built one thing.

**The "fit within the failover window" constraint is a design masterclass.** Every system has failure modes it already handles. Stripe's insight: make your new operation fit within an existing failure budget rather than trying to eliminate all disruption. This is a generalizable design principle that almost nobody follows. Most teams set "zero downtime" as the goal when "no worse than a failover" would be sufficient and dramatically simpler.

**The B-tree sort optimization reveals the value of deep infrastructure knowledge.** Morzaria's team understood MongoDB's storage engine well enough to know that sorting data by index attributes before bulk import would produce sequential B-tree writes. 10x improvement from understanding the layer below. This is the kind of optimization that generic cloud migration tools can never deliver — it requires owning and understanding the full stack.

**The custom MongoDB fork is both a strength and a liability.** Version gating patches and WAL tags are modest changes, but every MongoDB upgrade now requires forward-porting those patches. Stripe can afford this (they have the team, they built the upgrade platform), but it's a tax on every future MongoDB release. The build-vs-buy calculus needs to account for the maintenance tail, not just the initial build cost.

**What's conspicuously absent:** No discussion of failure modes during migration. What happens when the bulk import fails at 90%? When the coordinator crashes mid-fencing? When the oplog replication falls behind faster than it can catch up? A platform handling $1.4T in payments has answers to these questions, and their absence is frustrating. The talk is architecture, not operations — and operations is where databases earn their reliability.

**Comparison with [[PgDog]]:** PgDog provides proxy-layer sharding for Postgres with range-based keys and two-phase commit for cross-shard writes. DocDB does similar things for MongoDB but with a custom proxy and an actual data movement platform underneath. PgDog asks you to configure sharding; DocDB lets you change it at runtime. The difference is the data movement primitive.

**Comparison with [[Postgres CDC in ClickHouse, A Year in Review]]:** Both systems depend on CDC, but for opposite purposes. PeerDB/ClickPipes uses CDC for analytical replication; DocDB uses it for live data migration. PeerDB's "make it boring" framing applies equally here — DocDB's migration platform is impressive architecture, but the real engineering is in making it boring enough to run on $1.4T of payments.

**The Stripe double feature:** Between this and [[Minions — Stripe's One-Shot Coding Agents]], Stripe is publishing the most substantive large-scale infrastructure writing in the industry. Both pieces share a pattern: build a platform that turns an operation into a primitive. Minions makes code generation a primitive; DocDB makes data movement a primitive. The engineering philosophy is the same.

---

*Sources: [[summary/docdb-online-database]]*
*Last updated: 2026-05-18*
