---
url: https://tlockney.github.io/celld-book/
date_fetched: 2026-10-09
---

# celld: Durable Objects on Your Own Storage


---

<!-- https://tlockney.github.io/celld-book/introduction.html -->

## What this book is

Cloudflare's Durable Objects gave serverless programming a primitive it had been missing: a named object with one thread and its own storage, reachable from anywhere by its name. celld, from the Deno team, runs that same programming model on machines you control, with an object-storage bucket as the only coordinator. This book explains celld v0.6.0, the release that made it a beta. It covers what celld is, how it keeps its guarantees, how to build and operate an application on it, and where it still differs from the platform it imitates.

It is written for engineers who build stateful services and want to understand a system before they depend on it. You do not need to know Durable Objects already. You should be comfortable reading TypeScript, a shell session, and a little SQL.

## How it is organized

- **Part I, Foundations**(Chapter 1), is the background the rest assumes: the Cloudflare Workers platform, what a Durable Object is, the fifty-year lineage of the actor model, and durable execution.
- **Part II, How celld works**(Chapters 2–8), is the reference: the cell model, the bucket as coordinator, ownership and fencing, running a fleet, the Cloudflare compatibility surface, designing around cells, and a closing assessment.
- **Part III, Building on celld**(Chapter 9), is a walkthrough in ten steps, from an empty directory to an operated fleet.
- **Part IV, Labs**(Chapters 10–12), are executed notebooks. Each one drives a real- `celld dev`node and checks the book's claims against what the node actually does.
- **The appendices**hold a quick reference and self-quiz, the glossary, the release notes with every upgrade rule, and a look under the hood at SQLite's write-ahead log, Litestream, and the LTX files celld replicates.

## How to read it

Read Part I if Durable Objects are new to you; otherwise start at Part II. Part III can be read on its own, but it points back to Part II wherever the reasoning lives there. The labs are meant to sit beside Chapter 9: Lab 1 with Steps 02–04, Lab 2 with Steps 05 and 07, and Lab 3 with Step 06.

The labs are evidence, not illustrations. Every output in them came from a live node, and each lab is a frozen export of a run against celld v0.6.0 on 2026·09·26. Each ends with an exercise: TODO stubs and a checker cell. The checkers in the export print ✗ by design, because they are waiting for your solution. To run a lab yourself, download it from the top of its chapter together with the helper `celld_nb.ts` it imports, or get all three labs and the helper as one zip, and open it in JupyterLab with a Deno kernel.

A few conventions run throughout:

- **Section numbers**sit in the left margin of every chapter, and cross-references use them ("Chapter 4", "Chapter 9, Step 05", "Chapter 1 § 03").
- **Callouts**mark tips, warnings, and caveats in the margin by kind.
- **Terms**with a dotted underline have a definition on hover or tap. Appendix B collects them all.
- **Versions matter.**celld moves fast, with five releases in the four weeks before this edition. Where behavior changed between releases, the chapters state the current behavior, and Appendix C records the history once.

## How it was made

This book was generated with AI, from materials curated by Thomas Lockney for this purpose: celld's own documentation and release notes, Cloudflare's documentation and engineering posts, and a set of articles on actors and durable execution. The Colophon describes the process, and the Bibliography lists every source. Treat it as a well-researched guide rather than the vendor's documentation, and check anything load-bearing against the release you run.


---

<!-- https://tlockney.github.io/celld-book/foundations.html -->

## Why this part exists

celld is a self-hosted runtime for one specific programming model: Cloudflare Workers with Durable Objects at the stateful core. The rest of the book takes that model as given. Part II explains how celld implements it, Chapter 9 builds an application on it, and the notebooks check its behavior against a live node. None of them stop to say what a Durable Object *is*, why anyone would want one, or which older ideas it inherits.

This part fills that gap. It covers four things, in the order they build on each other:

- **The Workers platform**(§ 02): stateless JavaScript in V8 isolates, reached through bindings, deployed with Wrangler.
- **Durable Objects**(§ 03): a named, single-threaded object with its own storage, and the gates that make it safe.
- **The actor model**(§ 04): the fifty-year lineage from Hewitt to Erlang, Akka, and Orleans' virtual actors, of which a Durable Object is one descendant.
- **Durable execution**(§ 05 and § 06): code that survives crashes by replaying a log, and the distinction between an- *entity*that lives indefinitely and a- *process*that finishes.

§ 07 maps each idea onto celld's own vocabulary. Nothing here is specific to a celld release.

## The Workers platform

A Cloudflare Worker is a JavaScript (or WebAssembly) program that answers HTTP requests. It runs on Cloudflare's network rather than on a server you manage, and it keeps no state between requests of its own. Three pieces define the platform: the isolate it runs in, the bindings it reaches the rest of the platform through, and the tool that deploys it.

### Isolates, not containers

Workers run on **V8**, the JavaScript engine in Chromium and Node.js. In V8, an **isolate** is a lightweight sandbox: it gives one program its own variables and its own memory, safe from every other program in the same process. A single Workers runtime process hosts hundreds or thousands of isolates and switches between them with near-zero latency.

That choice is the platform's founding tradeoff. A container or virtual machine pays for an operating system and a language runtime every time an instance starts. Workers pays the runtime's cost once, when the host process starts. Cloudflare's documentation puts an isolate's startup at roughly a hundred times faster than a Node process in a container or VM, and its memory at an order of magnitude less, which is why one machine can run thousands of tenants' code with no per-tenant process.

When a request reaches any Cloudflare location, the machine that receives it runs the Worker's `fetch()` handler in an isolate on that machine. Requests are spread across the network wherever they land. That is exactly right for stateless code, and exactly wrong for anything that must coordinate: two requests for the same chat room can run on different continents.

### Bindings

A **binding** is a permission and an API in one object. It lets a Worker use a platform resource (a database, a bucket, another Worker) with no REST call and no credential in the code. The runtime holds the authorization, so the secret never reaches the script. Bindings appear on the `env` object, reached three ways:

- as an argument to a handler: `fetch(request, env, ctx)`;
- as `this.env`on a`WorkerEntrypoint`,`DurableObject`, or`Workflow`class;
- as an import for top-level code: `import { env } from "cloudflare:workers"`.

| Family | Bindings | 
|---|---|
| Storage and databases | Workers KV, R2 object storage, D1, Durable Objects, Hyperdrive, Vectorize, static assets | 
| Compute and communication | Service bindings (HTTP and RPC via `WorkerEntrypoint`), Queues, Workflows, Dynamic Worker Loaders, dispatchers (Workers for Platforms) | 
| Configuration | environment variables, secrets, Secrets Store, version metadata | 
| Platform services | Workers AI, Analytics Engine, Browser Run, Images, Stream, rate limiting, mTLS | 

celld supplies the bindings Cloudflare builds on Durable Objects (Durable Objects themselves, KV, Queues, D1, R2, Workflows, service bindings, Dynamic Workers, static assets, variables) and not the platform services; Chapter 6 draws the exact line.

### Wrangler

**Wrangler** is Cloudflare's command-line tool. A project declares its entry point, bindings, and Durable Object classes in a Wrangler configuration file (`wrangler.jsonc`, `wrangler.json`, or `wrangler.toml`), runs locally with `wrangler dev`, and ships with `wrangler deploy`. The configuration file is the contract between code and platform, which is why celld reads the same file (in its JSON forms; Chapter 6 lists the keys).

## Durable Objects

A **Durable Object** is the platform's answer to coordination. It gives a piece of state one home, one thread, and its own storage, and it routes every request for that state to that home.

### Why they exist

Before Durable Objects, a Worker had two places to keep state. It could call back to a central database at an origin, giving up the point of running at the edge. Or it could use Workers KV, which is built for read-heavy data and is eventually consistent, with last-write-wins semantics. Neither fits a chat room, a collaborative document, a game lobby, or a rate limiter: workloads that need many clients to agree on one current state, now.

The model arrived in stages, each announced on Cloudflare's blog:

| Date | Milestone | 
|---|---|
| 28 September 2020 | Kenton Varda announces Durable Objects in closed beta ("Workers Durable Objects Beta: A New Approach to Stateful Serverless"). | 
| 3 August 2021 | "Durable Objects: Easy, Fast, Correct—Choose three" introduces input and output gates. | 
| 15 November 2021 | Durable Objects become generally available. | 
| 11 May 2022 | Alarms launch ("Durable Objects Alarms—a wake-up call for your applications"). | 
| 5 April 2024 | JavaScript-native RPC replaces hand-written HTTP between a Worker and an object's stub. | 
| 26 September 2024 | "Zero-latency SQLite storage in every Durable Object" puts a SQLite database inside each object. | 

Boris Tane's one-line summary is the most useful starting point: *"A Durable Object is like having a tiny, long-lived server that is guaranteed to be unique for a specific ID."* A Worker spreads a hundred requests across whatever machines receive them; a Durable Object gathers every request for its ID into one place.

### One object per name, one thread

A Durable Object is an instance of a class in your code. Each instance has a globally unique ID, either derived from a name with `idFromName()` or generated by the system. An object with a given ID exists in **one place in the world at a time**. Any Worker, anywhere, that addresses that ID reaches that one instance: it gets a stub from the namespace binding and calls the object's methods over RPC, or forwards a request to its `fetch()`.

Cloudflare places the object near where it is first requested, or where a caller's location hint says. To reach it, the runtime asks an internal directory which machine currently runs that ID and routes the call there. If the machine fails, the object is started on a healthy one and its state is restored from storage. The caller never learns which machine holds the object.

Each object runs on **one thread**, in one isolate. All requests for that object go through that thread, so code can keep values in ordinary JavaScript variables across requests and maintain invariants without locks. This is the property the whole model rests on, and the one celld keeps exactly (Chapter 2).

### Storage and the two gates

Each object has private, strongly consistent storage. The original API is a key-value store. The SQLite-backed API embeds a SQLite database in the object's own thread, so a query runs synchronously, with no network round trip and no context switch.

The machinery underneath, which Cloudflare calls the **Storage Relay Service**, is worth knowing because celld rebuilds it on different parts (§ 07):

- SQLite runs in write-ahead-log mode, and a virtual-file-system hook captures each commit's WAL frames.
- The frames stream to **five follower machines in five different data centers**. When**three of the five**have them, the commit counts as durable.
- Every 10 seconds or 16 MB, the frames are batched and uploaded to object storage, and the followers drop their copies. If the object's machine dies first, the followers upload on its behalf.
- Object storage keeps the change log and periodic snapshots for 30 days, so an object can be restored to any point in that window.

D1, Cloudflare's SQL database product, is built on the same foundation: each database is a SQLite-backed Durable Object, and read replicas are further objects fed from the primary's WAL.

A single thread is not enough on its own, because JavaScript is asynchronous. Every `await` yields, and another request can start while the first one waits. The 2021 post introduced two mechanisms that close the gap without explicit transactions:

- **Input gate.**While a storage operation is in flight, the runtime delivers no new event (no new request, no WebSocket message) to the object. A read followed by a write cannot be interleaved by another request's read-modify-write.
- **Output gate.**Code does not have to wait for a write to reach disk. It continues immediately, and the runtime holds every outgoing message (the response, an outbound- `fetch()`) until the write is confirmed durable (with SQLite storage, until three followers hold it). If the write fails, the held messages are replaced by errors and the object restarts. No one outside the object can ever see a success for a write that was not persisted.

celld keeps the output gate by name and defines it against its own durability proofs (Chapter 4). It gets the input gate's effect a different way: storage calls are synchronous, so a storage operation never interleaves at all (Chapter 2).

### Hibernation and alarms

An object that is doing nothing should cost nothing. The **WebSocket Hibernation API** lets an object that serves WebSockets be evicted from memory while its clients stay connected at the edge. When a message arrives, the runtime re-creates the object (running its constructor again) and calls `webSocketMessage()`. In-memory variables do not survive; per-connection state can ride along with `ws.serializeAttachment()` (up to 16 KB) and come back through `ws.deserializeAttachment()`. Protocol-level ping and pong are answered at the edge without waking the object.

An **alarm** is the object's own timer. `this.ctx.storage.setAlarm(time)` schedules a wake-up; at that time the runtime calls the object's `alarm()` method. An object has one alarm at a time. Alarms run at least once: a throwing handler is retried with exponential backoff, starting at a 2-second delay, up to 6 retries. Alarms are the building block for background work, queues, and workflow engines.

## The actor model

A Durable Object is a new implementation of an old idea. The idea is the **actor**: a unit of computation with private state that interacts with the world only by exchanging messages. Knowing the lineage explains both the model's strengths and the specific choices Durable Objects made.

### Hewitt, 1973

Carl Hewitt, with Peter Bishop and Richard Steiger, introduced the actor model in **1973** as a mathematical model of concurrent computation. An actor is the fundamental unit. In response to a message, an actor can do three things, concurrently:

- send a finite number of messages to other actors;
- create a finite number of new actors;
- designate the behavior to use for the next message it receives.

An actor's state is private, and actors communicate only by asynchronous messages. No actor reaches into another's memory, so there is nothing to lock.

David Khourshid's everyday version: an actor is a coworker you message on Slack. They read the message when they are ready, may do some work or update their to-do list, and may message you or others back. *"You never reach over and edit their work directly (hopefully), and you don't know what they're thinking unless they tell you."* Every actor has three parts: a **mailbox** of incoming messages, **private state** nothing else can change, and a **behavior**, which he writes as *state + message → next state (+ effects)*. That last form is why state machines fit actors so naturally, and his explanation for why the model keeps being reinvented, most recently for AI agents that hold their own context and message each other: *"We keep reinventing it because it keeps being right."*

### Erlang and "let it crash"

Joe Armstrong, Robert Virding, and Mike Williams built **Erlang** at Ericsson in **1986** to program telephone exchanges; it was released as open source in December 1998. An Erlang program is a large number of very lightweight processes on the BEAM virtual machine. Processes share no memory, collect their own garbage, and communicate by asynchronous, location-transparent messages delivered to mailboxes.

Erlang's contribution to fault tolerance is **let it crash**. A process does not defend against every unexpected error. It fails cleanly, and a separate process restarts it from a known state. **OTP** organizes this into **supervision trees**: supervisor processes own worker processes, are notified when one dies, and apply a restart strategy, so a fault is contained without taking the node down.

### Akka

**Akka** brought the Erlang model to the JVM, for Java and Scala. An actor processes messages from its mailbox one at a time, replacing shared-memory threading with sequential message handling. Every actor is supervised by its parent, as in OTP, and the model extends across a cluster.

### Orleans and the virtual actor

Classic actors have a lifecycle the programmer must manage. Someone creates the actor, holds its address, and handles its disappearance when its host dies. Microsoft Research's Orleans removed that burden. The paper "Orleans: Distributed Virtual Actors for Programmability and Scalability" (Bernstein, Bykov, Geller, Kliot, and Thelin; technical report MSR-TR-2014-41, **March 2014**) introduced the **virtual actor**, which Orleans calls a **grain**:

- **Perpetual existence.**A grain exists logically at all times. It is never explicitly created or destroyed, and a server crash does not end its identity.
- **Automatic activation.**The runtime loads a grain into memory when a message arrives for it, and deactivates it when it is idle.
- **Location transparency.**A caller addresses a grain by identity and never needs to know which server (a- **silo**) is hosting it.

A grain's durable state lives in an external storage provider, such as a SQL database, Cosmos DB, or Redis.

### Where Durable Objects sit

A Durable Object is a virtual actor. It has a global identity, it is created implicitly on first use and evicted when idle, it runs one message at a time, and callers reach it through a stub without knowing where it lives. Four things set it apart:

| Classic actor (Erlang, Akka) | Virtual actor (Orleans grain) | Durable Object | |
|---|---|---|---|
| Lifecycle | created and stopped explicitly; gone if its host dies | perpetual; activated and deactivated by the runtime | perpetual; created on first use, hibernated or evicted when idle | 
| Addressing | a process or network address | a logical identity | a globally unique ID or name | 
| Durable state | the programmer's problem | an external storage provider | colocatedstorage in the same thread (key-value or SQLite) | 
| Concurrency | one message at a time | one message at a time | one thread, plus input and output gatesaround storage and output | 
| Host | a VM or JVM cluster | .NET silos | V8 isolates on Cloudflare's network, with hibernating WebSockets | 

The colocated storage and the gates are the decisive differences. An Orleans grain must call out to its database, and that call is where latency and consistency bugs come from. A Durable Object's database is in its own thread, and the output gate makes durability invisible to the programmer. celld keeps both properties and changes only where the object runs and where its storage is replicated (§ 07).

## Durable execution

Actors solve *where state lives and who may change it*. A second family of systems solves a different problem: *how a multi-step process finishes even when the machine running it does not*. That property is **durable execution**.

### The idea

An ordinary process keeps its progress in memory; when it crashes, the progress is gone. A durable execution engine records each step's outcome in a persistent log. When the process dies, a new process picks the execution up, and the code continues as if the crash never happened, with its local variables and its position restored. A durable execution can last a fraction of a second or wait for months.

The mechanism, in nearly every engine, is **replay**. The engine runs the code again from the start. Each time the code reaches a step that already completed, the engine returns the recorded result instead of running the step. When the code reaches the first step with no record, it is back where it crashed, and real execution resumes.

Replay imposes two rules on the programmer. Jack Vanlightly's "Demystifying determinism in durable execution" (2025) draws the line precisely:

- **Control flow must be deterministic.**The branches, the loops, and- *the arguments passed to each side effect*must come out the same on every replay, or replay drifts onto a different path. A clock read, an unseeded random number, or a direct database query in the control flow can cause it; the result is bugs like charging a customer twice.
- **Side effects do not need to be deterministic, but they must tolerate running twice.**An API call or an email send goes inside a step. The engine records the step's result once it completes and returns the record on replay. A crash- *after*the side effect but- *before*the record is written runs the step again, so the step must be idempotent.

Stripped down, durable execution is **persistent memoization**. Gunnar Morling showed how little it takes in "Building a durable execution engine with SQLite" (2025): a working engine in under 1,000 lines of Java, with one SQLite table (`execution_log`) recording each step's status, parameters, and return value, a proxy that intercepts step calls to consult and update the log, and virtual threads to park a flow that is sleeping or waiting for a signal. A durable log, an interceptor, and a way to suspend: every engine below is some arrangement of those three parts.

### Five engines

| Engine | How it records progress | How it recovers | 
|---|---|---|
| Temporal | an append-only event history of every activity, timer, and result | re-executes the workflow code from the beginning, answering completed actions from the history | 
| Restate | a journal of every step, RPC call, timer, and state update, kept by the Restate server (written in Rust); state lives beside the journal in an embedded key-value store | replays the journal, skipping completed steps | 
| Azure Durable Functions | event sourcing through the Durable Task framework, with orchestration history in Azure Storage or the Durable Task Scheduler | the orchestrator replays its history to rebuild its state | 
| DBOS | an in-process library (DBOS Transact, for Python, TypeScript, Go, Java, and Rust) that writes each workflow's status and inputs, then each step's output, to tables in the application's Postgres database | on restart, a background thread finds workflows still `PENDING`and calls each again from the start with its original inputs; completed steps return their recorded outputs | 
| Cloudflare Workflows | each workflow instance's engine is a SQLite-backed Durable Object; `step.do()`results are stored in its SQLite database | `run()`executes again and each completed`step.do()`returns its stored result;`step.sleep()`and retries wake the engine with Durable Object alarms | 

The engines also differ in *where* they run. Temporal is an **external orchestrator**: a separate server cluster, with workers that talk to it over the network. DBOS argues for the opposite, **lightweight durable execution**: an in-process library that checkpoints each step into the application's own database (Postgres, in DBOS's case), with no orchestrator to deploy. Cloudflare Workflows sits between the two. The engine is not a server you run, but each instance is a Durable Object: Morling's three parts, with the log in the object's SQLite and alarms as the way to suspend.

Vendors claim more than crash-proofing for the model. Alex Poliakov (DBOS) lists observability (every step's inputs and outputs are already recorded), forking (re-running a failed workflow from a chosen step on fixed code), and workflows that run for months across restarts and deployments.

Cloudflare announced Workflows in open beta on **24 October 2024** ("Build durable applications on Cloudflare Workers: you write the Workflows, we take care of the rest"). It is the case this book cares about most, because it is built out of Durable Objects: the process primitive is implemented on top of the entity primitive. celld's Workflows implementation follows the same design (Chapter 6, "Workflows"; Lab 3).

## Entities and processes

Put § 04 and § 05 side by side and two kinds of stateful thing appear:

- An **entity**is a named unit whose state persists indefinitely: a user, a room, a document, a device. It has no natural end. It handles one operation at a time, forever.
- A **process**is a sequence of steps that starts, runs, and**finishes**: an order pipeline, a signup flow, a nightly import.

Azure Durable Functions draws the line most explicitly. Alongside orchestrations it offers **Durable Entities** (added in Durable Functions 2.0), its version of a virtual actor:

| Durable Entity | Orchestration | |
|---|---|---|
| Purpose | explicit, long-lived state (a counter, a cart, a session) | a sequence of tasks that runs to completion | 
| State | held explicitly and changed by operation handlers | implicit in the code's control flow and its history | 
| Lifecycle | created on first use, unloaded when idle, persists indefinitely | started by a trigger, finishes when done | 
| Execution | operations processed serially, one at a time | replays history to drive its activities | 
| Identity | an entity ID: an entity name plus an entity key | an instance ID | 

Azure also shows the price of separating them. An entity can *signal* another entity (one-way, no reply), but only an orchestration can *call* one and wait for the answer, and only an orchestration can coordinate several entities at once. Compared with an Orleans grain, a Durable Entity favors durability over latency: it persists its state on every operation.

The Durable Objects platform has both shapes, and builds one from the other. A Durable Object is the entity. Workflows is the process, and each workflow instance runs on a Durable Object. The practical rule follows directly: **model the thing that persists as an object, model the thing that finishes as a workflow**, and do not force one shape into the other. Chapter 7 ("Entities, not processes") applies this rule to application design.

## Where celld fits

| In this part | In celld | Where the book covers it | 
|---|---|---|
| a Durable Object instance | a cell | Chapter 2 | 
| Cloudflare picks a data center | an ownernode claims the cell'sownership recordin the bucket | Chapters 3 and 4 | 
| Storage Relay Service: WAL frames to five followers, durable at three, batched to object storage | SQLite replicated as LTX segmentsto thefleet bucket; acknowledged on afleet proof(one or two followers,everyfollower must fsync) or, on a single node, abucket proof | Chapter 3 | 
| the internal directory that locates an object | the cell's ownership recordin the bucket; no directory service | Chapters 3 and 4 | 
| D1 as SQLite-backed Durable Objects | D1 as a cell | Chapter 6 | 
| output gate | output gate, holding output until a durability proof covers it | Chapter 4 | 
| input gate | synchronous storage calls: a storage operation never interleaves | Chapter 2 | 
| hibernation and eviction | resident,hibernated, andinactivecells | Chapter 2; Lab 3 | 
| alarms | durable alarms, which also drive cron and Workflows | Chapter 6; Lab 1 | 
| Workflows (replay over `step.do()`) | Workflows on cells, with the same replay discipline | Chapter 6; Chapter 9, Step 06; Lab 3 | 
| `wrangler dev`/`wrangler deploy` | `celld dev`/`celld deploy`, reading the same`wrangler.jsonc` | Chapter 9, Steps 02–03, 09 | 

From here, read Part II for how celld makes these guarantees on commodity machines, Chapter 9 to build something, and the notebooks to watch it happen. Appendix B defines every term the book uses.

This chapter draws on: Cloudflare documentation (7 entries) · Cloudflare blog and talks (10 entries) · The actor model (6 entries) · Durable execution (10 entries). The full entries are in the Bibliography.


---

<!-- https://tlockney.github.io/celld-book/how-celld-works.html -->

celld is a stateful distributed system that runs server-side JavaScript on your machines and keeps all of its shared state in an S3-compatible, Google Cloud Storage, or Azure Blob bucket that you own. It is **V8 + SQLite + LTX**: the Cloudflare Workers runtime with Durable Objects as the stateful core, with the placement layer replaced by an object store. The JavaScript API it exposes is the same API that Workers and Durable Objects supply, so code written for one side generally runs on the other. celld's conformance tests run programs on workerd (the binary Cloudflare operates in production) and on celld, on identical bytes, and require equal output.

The founding idea is worth stating plainly. The Durable Objects model, *a single-threaded object with its own storage, addressed by name*, is one of the best primitives distributed systems has been handed in years. celld does not contest that. It moves placement, state, and operational evidence out of a shared vendor platform and into infrastructure you choose. The docs are explicit that self-hosting is not automatically more reliable; it makes the failure domain explicit and inspectable: your nodes, your bucket provider, and your operational choices.

If Durable Objects, actors, or durable execution are new to you, read Chapter 1 first: Actors, Durable Objects, and Durable Execution covers the platform and the ideas this part takes for granted.

This book describes celld v0.6.0 (released 2026·09·26), the first release celld calls a **beta** rather than an alpha. Where behavior changed between releases, the body states the current behavior and Appendix C records the change once, next to the upgrade rules.

Durable Objects is a strong programming model. celld keeps the model—a named object, one thread, its own SQLite—while moving the scheduler, the storage, and the failure domain onto your machines and your bucket.

This chapter draws on: celld: documentation at v0.6.0 (9 entries) · celld: release notes (5 entries) · Cloudflare documentation (4 entries). The full entries are in the Bibliography.


---

<!-- https://tlockney.github.io/celld-book/cell-model.html -->

A **cell** is the Durable Object: a small server with a name and a private SQLite database. You make one cell for each user, each document, each chat room, or each AI agent. A cell serves HTTP, holds WebSocket connections, sets alarms, and makes outbound connections. Two properties make the model safe without distributed locking:

- **One thread per cell.**Two requests to the same cell never run at the same instant. A second request can interleave only while the first- *awaits*, and storage operations are synchronous, so a storage operation never interleaves at all. The data in a cell stays consistent by construction. Lab 1, Part 2 shows it live: with a timer awaited between read and write, ten concurrent increments all return 1; wrapped in- `blockConcurrencyWhile()`, they return 1 through 10.
- **No shared database.**Cells share nothing; the application divides into cells from the start. The contention of one shared database never appears, because no shared database exists.

## Cell states

A cell has the same states as a Durable Object:

- **Resident**: in memory. A resident cell is- *active*while it does work and- *idle*while it waits.
- **Hibernated**: evicted from memory. Its hibernatable WebSocket clients stay connected and it stays on its node.
- **Inactive**: no node holds it. It is only an object in the bucket, at essentially zero cost.

Every cell starts inactive. One 8 GB node holds ~1,000 resident cells, which prices a resident cell at roughly $0.05/month. Idle eviction is opt-in: `CELLD_IDLE_EVICT_S` sets how many seconds without work send an idle resident cell to hibernation, and when it is unset only memory pressure or the residency cap removes one. That matters for balancing (Chapter 5), which moves only hibernated cells. Lab 3, Part 2 watches a chat room hibernate while both of its sockets stay open, then wake on the next message with its constructor running again.

Two practical consequences follow. First, **keep the constructor light**. It runs on every wake, including every message to a hibernated cell, so restore state from SQLite storage inside the handler, not in the constructor. Second, **idle cost is near zero**: an agent fleet whose cells hibernate between events costs almost nothing to hold, which is exactly the economics the model is designed for.

This chapter draws on: celld: documentation at v0.6.0 (9 entries) · celld: release notes (5 entries) · Cloudflare documentation (4 entries). The full entries are in the Bibliography.


---

<!-- https://tlockney.github.io/celld-book/bucket.html -->

This is the architectural centerpiece. There is **no membership protocol, no failure detector, and no consensus service**. A **node** is one celld process; a **fleet** is the set of nodes sharing one **fleet bucket**. Ownership of a cell is a record in that bucket, claimed with one atomic write. celld's built-in replicator continuously ships each cell's SQLite state to the bucket as **LTX segments**, Litestream's replica format from Ben Johnson. The loss of a node cannot lose an acknowledged write, because celld does not answer a write until the data survives a failure (**RPO=0**).

## How RPO=0 is earned: bucket proof vs. fleet proof

The durability mechanism depends on fleet size, and it is explicit.

- **With one node**, every write waits for the bucket. This is the- **bucket proof**: one storage round trip, which is the minimum latency for a durable write.
- **With two or more nodes**, the node serving the cell sends each write to another node and answers as soon as that node has the data on its own disk; the bucket upload finishes afterwards. This is the- **fleet proof**. The owner and the nodes it sends to form the cell's- **ensemble**, and each of those nodes is a- **follower**. A node picks one or two followers (never itself), so a fleet of three or more nodes holds- *three copies*of an acknowledged write, and the ensemble keeps acknowledging while one follower remains.

`CELLD_DURABILITY` selects the mode (default `fleet`). A single node requests the fleet posture and does not get it, falling back to bucket proof. Run two or more nodes if write latency matters.

Four properties make this possible, and they are non-negotiable requirements on the store:

- **Conditional create**: creating an ownership record fails when the object already exists.
- **Conditional overwrite**: a compare-and-swap on the prior record fails when the object changed after the read.
- **Read-after-write consistency**: a read after a successful write returns that write.
- **Exact ranged reads**: a- `Range`request returns precisely the requested bytes. A large cell is restored- *page by page*through a fault-in SQLite VFS that reads each page from the bucket on first use, so a wrong range is a correctness failure, and the startup probe checks it.

| Store | Fleet-qualified? | Conditional-write path | 
|---|---|---|
| Amazon S3 | Yes | `If-None-Match: *`/`If-Match`etag CAS | 
| Cloudflare R2 | Yes | same headers; celld's release tests run here | 
| Google Cloud Storage | Yes | XML API `x-goog-if-generation-match` | 
| Tigris | Yes | documented conditional operations | 
| Azure Blob Storage | Yes | `If-None-Match: *`/`If-Match`on Put Blob; qualified 2026-08-18 | 
| MinIO (community) | Passes test, not qualified | conditional writes work on RELEASE·2025-09-07 or later (#162 pinned one broken release) | 
| Backblaze B2 | No | — | 
| Hetzner Object Storage | No | — | 
| DigitalOcean Spaces | No | — | 

A bucket value can carry a key prefix (`s3://bucket/team-a`), so two fleets can share one bucket. A value without a prefix keeps objects at the bucket root, so an existing fleet never moves its data. The bucket also holds the deployments (with container images under `deploy/images/`), node leases, the shared peer-authentication secret, the fleet capacity sample, the alarm wake entries, large KV values, and all R2 objects: **whoever holds the bucket credentials controls the fleet**.

This chapter draws on: celld: documentation at v0.6.0 (9 entries) · celld: release notes (5 entries) · Cloudflare documentation (4 entries). The full entries are in the Bibliography.


---

<!-- https://tlockney.github.io/celld-book/ownership.html -->

celld makes two claims that a distributed system must actually prove: *one node owns a cell at a time*, and *a write is durable before it is acknowledged*. Both rest on the bucket and on epochs, not on clocks. The docs page What celld guarantees states the two promises up front and shows the mechanism.

## The ownership record and the epoch

Each cell has one **ownership record** in the bucket. It names the **owner** (the node session that may run the cell) and a fencing **epoch**. A node acquires a cell with a conditional write: create when no record exists, compare-and-swap when one does. The bucket accepts only one such write, so two nodes cannot acquire the same cell. Every activation advances the epoch: a takeover advances it, and a local wake advances it too. The replicator writes each cell's SQLite data under an epoch prefix, `cells/<cell>/ltx/e<epoch>/`.

## The acknowledgement rule (RPO=0)

The **output gate** holds each write response until a durability proof covers it. After a *bucket proof*, celld re-reads the ownership record and acknowledges only if it still names this node at this epoch. A partitioned node can commit locally and replicate into its superseded prefix, but the ownership read reveals the new owner, so the write is not acknowledged. The check reads the record rather than comparing a clock, so a paused process or a skewed clock cannot pass it. A *fleet proof* requires every follower to fsync the write, and a takeover seals the prior node-log session before restoring, so the stale owner cannot complete another fleet proof. The output gate applies one ordering rule across responses, outbound calls, Queue deliveries, and WebSocket sends, and read-only output waits for earlier request or alarm writes to become durable.

## Self-fencing

Each node holds a **node lease** in the bucket with an expiry (`CELLD_TTL_MS`, default 10,000 ms), renewed after one third of the lifetime. A node that cannot reach the bucket cannot renew or replicate, so it must not own cells. When its published expiry passes it **fences itself**: it stops each active cell, fails incomplete requests, logs a line starting `SELF-FENCE:`, and exits with code 3. The fence names its cause with a distinct event:

- `node_lease_watchdog_fence`: the lease expired.
- `node_lease_record_missing_fence`: the record is gone.
- `node_lease_record_mismatch_fence`: another writer replaced it. The node cannot prove who, so it names no author.

A failed renewal retries before the authority expires. `RUST_LOG=celld=info,store=debug` logs every lease read and write with its outcome, at no cost when off. The fenced state is terminal; only a restart returns the node to the fleet. Two requirements follow:

- **Run under a supervisor**(systemd, Docker restart policy, Kubernetes) with no attempt limit, waiting at least one lease lifetime between attempts. A node that cannot acquire a lease at startup retries rather than exiting. A restarting node first recovers its previous session's log before it takes a lease. If a peer is already recovering that log, it waits behind the peer's heartbeat and takes over only when the heartbeat stops, which is what makes a whole-fleet restart recover every acknowledged write.
- **A request is refused before the fence runs.**celld compares the current time against the published expiry on every route, so a node with a lapsed lease refuses the request. The dispatch check keeps one owner per cell even while the fence is in flight.

## Remote calls and retries

Every proxied cell call (fetch, RPC, and WebSocket) runs through one versioned peer tunnel that streams the request body to the owner. The consequence for application code: **celld does not retry a call after transmission begins**, because it keeps no replay copy of the body. It retries only a peer attempt that proves the handler did not start; an ambiguous attempt (the handler may have completed without returning) is not retried. A stale route, a call that resolves an owner generation a replacement process now rejects, is handled underneath: celld waits for a different (node, epoch), refreshes the route, and re-attempts within `CELLD_OPERATION_DEADLINE_MS`. Keep one stable operation ID when you retry an ambiguous fetch, RPC, D1, or service operation, and make operations idempotent at the application layer. An `AbortSignal` passes through an RPC call on the same node; it does not cross a node boundary.

This chapter draws on: celld: documentation at v0.6.0 (9 entries) · celld: release notes (5 entries) · Cloudflare documentation (4 entries). The full entries are in the Bibliography.


---

<!-- https://tlockney.github.io/celld-book/fleet.html -->

## Install and storage configuration

The installer downloads a signed binary. Replication runs in the celld process, so no external replicator is needed. Pin exact releases with `CELLD_VERSION` and verify build attestations with `gh attestation verify`. The immutable releases sit behind a single `current` pointer, which makes a previous SHA the rollback; there is no automatic update agent. Prebuilt binaries cover Linux x86-64, Linux ARM64, and Apple Silicon; Windows is not supported. On Amazon EKS, celld reads Pod Identity credentials from the injected environment and token file.

```
# one-time install (pin a release with CELLD_VERSION)
curl -fsSL https://celld.dev/install.sh | sh
# Cloudflare R2 bucket — the standard AWS credential chain works
export AWS_ACCESS_KEY_ID=...
export AWS_SECRET_ACCESS_KEY=...
export AWS_REGION=auto
export S3_ENDPOINT=https://ACCOUNT_ID.r2.cloudflarestorage.com
export CELLD_BUCKET=s3://cells
```
Three storage families, one environment contract:

- **S3-compatible**(- `s3://`): standard AWS chain; R2 via the S3 endpoint above; EKS Pod Identity supported.
- **Google Cloud Storage**(- `gs://`): Application Default Credentials or a- `GOOGLE_APPLICATION_CREDENTIALS`service-account key; no AWS variables, region ignored.
- **Azure Blob**(- `az://`): exactly one credential family, which is an account key, VM managed identity, or AKS workload identity; the bucket name is the container.

## Develop locally with celld dev

`celld dev` opens a **local store** (a local SQLite object store), deploys the application, and starts one node, with no Docker and no cloud bucket:

```
cd ./my-wrangler-project
celld dev              # Worker listener on http://127.0.0.1:9876 by default
celld dev --port 3000  # pick the port
celld dev --host 0.0.0.0  # expose the Worker listener (internal stays loopback)
celld dev --logs       # show the node's info/warning logs too
celld dev --clean      # discard .celld/dev first
celld dev --watch-ignore "docs/**"   # extra watcher ignores (repeatable)
celld dev --no-watch   # no automatic builds or restarts
```
State lives in `.celld/dev` under the project (add `.celld/` to `.gitignore`), survives normal shutdown, and resets with `--clean`. The command watches the project and adopts a rebuilt deployment automatically; a failed build leaves the current app serving. The watcher ignores `.celld`, `.wrangler`, `.git`, `node_modules`, `target`, and any `--watch-ignore` globs, and a read is not a change. `--no-watch` turns automatic builds and restarts off entirely and cannot be combined with `--watch-ignore`. `celld dev` also reads a `.dev.vars` file beside the Wrangler config, as `wrangler dev` does: `NAME=value` lines, quotes stripped, overriding same-named `vars`, reloaded on edit, and never shipped to a fleet. Worker projects need esbuild on `PATH`; asset-only and `no_bundle` projects do not. The local store is not selectable by fleet nodes or operator subcommands. Fleets require a qualified cloud bucket.

## Deploy an application

`celld deploy` runs from a Wrangler project and accepts module Workers, Durable Object bindings, static assets, service bindings, D1 databases, KV namespaces, Queues, R2 buckets, Workflows, WebAssembly modules, cron triggers, `worker_loaders`, and `containers`. It stops with a named error on any Wrangler key it does not model. esbuild on `PATH` is needed only for projects with Worker code. Two properties of the deploy itself protect the fleet from a bad build. The manifest records a full SHA-256 digest for every JavaScript and WebAssembly module, and a node verifies each one before it builds the deployment, so changed bytes cannot become active. A prebuilt project (`no_bundle: true`) has its entry JavaScript preserved byte-for-byte, and celld discovers `**/*.wasm` modules below the entry's directory using Wrangler's default patterns (symlinks refused, no `rules`/`find_additional_modules`).

A running node adopts a new deployment in place. It reads `deploy/current.json` every 30 seconds (`CELLD_DEPLOY_POLL_S`), builds the new deployment beside the one it serves, then switches new requests to it in one step; a request started on the previous deployment finishes on it. `POST /reload` on the internal listener adopts immediately. A Durable Object that is not resident runs the new deployment at its next activation; a resident one moves at a safe point. In the adoption window a request on one deployment can call a Durable Object on the other, so adjacent versions must accept each other's calls.

## Start nodes and grow the fleet

Local development needs only the default listener. A fleet node binds two listeners: a public Worker listener for ingress and an internal listener for the peer protocol and operator API. An explicit advertised address requires an explicit internal-listener address. celld also rejects an explicit non-loopback public listener without an internal one; this rule catches a stale single-listener configuration.

```
celld \
  --bucket "$CELLD_BUCKET" \
  --listen 0.0.0.0:8080 \
  --internal-listen 10.0.0.12:8081 \
  --advertise node-a.internal:8081
```
To add a node, point it at the same bucket with a distinct internal address. **There is no join command and no fixed membership list**: nodes find each other through the leases in the bucket. The bucket supplies discovery and authority; it does not supply network reachability. The peer tunnel carries versioned plain HTTP for cell fetch and RPC traffic, with no content signature, so the private network is the security boundary; the fleet HMAC authenticates tunnel establishment and control requests. celld does not terminate TLS. Put the advertised addresses on a private network or an encrypted overlay such as WireGuard or Tailscale, and never expose the internal listener.

## Ownership balancing

A joining node takes hibernated cells from the nodes holding the most, so it carries its share within minutes, and the fleet evens out again after a node leaves. The mechanism keeps the no-coordinator posture. Every node reads a shared fleet capacity sample every 5 s (`CELLD_REBALANCE_INTERVAL_MS`; `0` disables). One node claims the refresh with a conditional write, reads every lease, and writes `fleet/capacity-v1.json`, so the cost does not grow with the square of the fleet. Each node's target is the fleet's owned cells divided by weight (`CELLD_PLACEMENT_WEIGHT`, default the CPU count). The node with the most owned cells per unit weight hands at most 32 hibernated cells per sample to the peer furthest below its share, one ownership-record write and one signed acquire each, and the receiver fills to 2% below target.

Only hibernated cells move. A resident cell hibernates through idle eviction first (`CELLD_IDLE_EVICT_S`), so a fleet without idle eviction balances only the cells that hibernate on their own. A moved cell's parked WebSockets close with code 1012. A draining node, or one with an activation backlog, receives nothing. The fleet moves nothing while any lease lacks a weight, so a rolling upgrade completes before the first move. `POST /rebalance/pause` and `/resume` on any internal listener govern the whole fleet.

## Graceful shutdown and upgrades

SIGTERM/SIGINT (what `systemctl stop`, `docker stop`, and a Kubernetes pod delete send) triggers a graceful drain: `/.well-known/celld/health` reports unhealthy, new public requests get a 503, and the node hands resident cells to peers. The handoff is batched and heavily engineered. For each batch the node:

- reserves a batch of cells, ordered by local request count with the newest request ID breaking ties;
- stops new local routes;
- cancels firing alarms, arming a durable wake so the successor runs them at least once, and cancels any active internal fetch/RPC handler;
- proves the batch durable in the live ensemble;
- publishes a full L9 snapshot and verifies the bucket holds a restore object, so the successor skips replay (a database too large for the 10 s durability budget, roughly 80 MiB, skips the snapshot and hands off through its L0 chain);
- releases ownership and asks a compatible peer to acquire, waiting for each acknowledgement before the next batch.

One variable bounds the whole stop. `CELLD_SHUTDOWN_TOTAL_MS` (default 40,000) derives the fleet drain-token wait (3/4 of it, 30 s) and the no-progress bound (5/8, 25 s). `CELLD_RELEASES` sets the number of concurrent handoffs (default 128). A fresh process holds its first healthy response until the fleet is settled (`CELLD_READY_FLEET_GATE_MS`, default 120,000). If the gate expires, the node emits a `ready_gate_expired` event once and **keeps readiness closed** until the condition clears, so give the orchestrator a rollout deadline that fails a persistent capacity problem.

## Release notes

Appendix C records what each release changed, one row per release from v0.4.0 to v0.6.0.

## Diagnose a fleet

`celld diagnose` reads the node leases from the bucket and probes each live peer; it never takes a lease or changes ownership. It reports expired records, unsafe or incorrect advertised addresses, unreachable peers, authentication failures, and version disagreements, shows each node's load sample (owned cells, resident cells, WebSockets, RSS, CPU, file descriptors, pressure, shedding), and runs the storage test. `celld cell list` lists Durable Object instances, each line `Class:ID`. D1 databases, KV namespaces, and Workflows appear as reserved `__` cells. Paginate with `--after`; a class-name argument scopes the storage prefix so `--limit` applies to that class alone, and the listing does not load the namespace into memory. During a rolling update, wait for every node to report `restoring=0` before restarting the next.

`GET /state` on the internal listener is an autoscaler feed. It reports `owned_cells`, `occupied`, `capacity_waiting`, `activation_waiting`, `restoring`, and `shedding`, counters for `handed_off`, `rebalanced`, `rebalance_failed`, and `remote_route_refreshes`, and a `node_load` object mirroring the lease's sample (`placement_weight`, `resident_cells`, `host_websockets`, `rss_bytes`, `cpu_percent_x100`, `open_fds`, `pressured`, `memory_headroom`). It also carries `allocator` and (Linux) `libc_malloc` memory counters and a per-script `deployment.isolates` block: `live`, `live_empty`, `retiring`, `freed`, V8 heap and external bytes. A `live_empty` count that persists past 30 seconds signals a stuck maintenance pass, and a persistent `retiring` count a stuck request. A positive `capacity_waiting` is the add-a-node signal; scale down only while every remaining node reports headroom and a small `restoring` backlog. The health path stays a plain boolean: 503 during drain and before settle, no utilization number. `/evict/<cell>` waits for the eviction and answers `{"ok":true}`, or `{"ok":false,"error":{"kind":…,"reason":…}}` with a kind of `refused` (409/503: `cell_active`, `alarm_imminent`, `eviction_limit`, …), `cancelled` (new activity or a fence), or `failed` (500: a lost reply or a durability failure).

## Operate D1, KV, Queues, and R2

`celld d1` runs SQL and migrations against a deployed D1 database, routing through the fleet to the database cell. The migration extension is ASCII case-insensitive, and `migrations_dir` must be a relative path inside the project. `celld kv` reads and writes a deployed KV namespace. The bulk commands use the Wrangler file format, so `wrangler kv bulk get` can export data for `celld kv bulk put`, and `celld kv bulk get` streams rows rather than holding the namespace in memory (an `expiration` is exported in Unix seconds, as Wrangler expects). `celld kv list` caps at 1000 keys per read and reports `--after` to continue. Every celld command writes data to stdout and messages to stderr, so redirects and pipes carry only data. `celld queue info/peek/purge/pause/resume/redrive` operates queues (`purge` needs `--force`; `peek` and `redrive` take `--limit` 1–100). `celld r2 get|head|put|delete|list` stands in for `wrangler r2 object`: it reads the fleet bucket directly with no running node, takes the `bucket_name` rather than the binding name, streams a `get` to stdout, and preserves the binding's metadata (`--content-type`, `--cache-control`, `--metadata JSON`, …); `--local`, `--remote`, and `--jurisdiction` are refused with an explanation.

## Primary environment variables

| Variable | Purpose | 
|---|---|
| `CELLD_BUCKET` | Fleet bucket (+ optional key prefix). Same as `--bucket`. | 
| `S3_ENDPOINT`,`AWS_REGION`,`AWS_*` | S3-compatible endpoint and credentials (standard AWS chain; EKS Pod Identity supported). | 
| `GOOGLE_*` | Google credentials for a `gs://`bucket (ADC or service-account key). | 
| `AZURE_*` | Azure account/identity for an `az://`bucket (exactly one credential family). | 
| `CELLD_DURABILITY` | Durability mode: `fleet`(default) or`bucket`. Fleet proof needs ≥ 2 nodes. | 
| `CELLD_ADDR`/`CELLD_INTERNAL_ADDR`/`CELLD_ADVERTISE` | Public listener, internal peer/operator listener, advertised address. | 
| `CELLD_ACTIVATIONS` | Concurrent cold-cell activations (default 8 per CPU, at least 16 and at most 128; a cold activation mostly waits on the store, so the default sits above the CPU count). | 
| `CELLD_MAX_RESIDENT_CELLS` | Hard resident-cell cap, enforced at admission. | 
| `CELLD_IDLE_EVICT_S` | Seconds without work after which an idle resident cell hibernates (unset: only pressure or the cap removes it). Balancing moves hibernated cells only. | 
| `CELLD_PLACEMENT_WEIGHT`/`CELLD_REBALANCE_INTERVAL_MS` | This node's ownership share relative to its peers (default: CPU count) and the fleet-sample interval (default 5,000; 0 disables balancing). | 
| `CELLD_MAX_CELL_REQUESTS` | Concurrent fetch limit for one Durable Object (default 64). A Queue broker has its own fixed limits: 256 concurrent producer calls, 64 per transaction, four overlapping proofs. | 
| `CELLD_MAX_REQUEST_BODY_BYTES` | Body limit for a public Worker request or direct DO request (default 1 GiB). | 
| `CELLD_MAX_RSS_MB` | Memory threshold for pressure shedding (default 80% of available memory; accounts for cgroup memory on Linux). | 
| `CELLD_TTL_MS` | Node-lease lifetime (default 10,000 ms). | 
| `CELLD_OPERATION_DEADLINE_MS` | Deadline for a non-restore operation (default 15,000). | 
| `CELLD_DEPLOY_POLL_S`/`CELLD_DEPLOY_MAX_AGE_S` | Deployment-adoption poll interval (30s) and forced-move age (60s). | 
| `CELLD_SHUTDOWN_TOTAL_MS`/`CELLD_RELEASES` | Total stop bound (40s; the drain-token wait and no-progress bound derive from it) and concurrent handoffs (default 128). | 
| `CELLD_LTX_COMPACTION` | 1 (default) creates additive L1 objects so a takeover reads tens of objects instead of thousands. | 
| `CELLD_LTX_PAGED`/`CELLD_LTX_PAGED_MIN_MB`/`CELLD_LTX_HYDRATE_MBPS` | Paged restore (default on) for chains above the threshold (default 256 MiB), and the background fill rate for the paged file (default 16; 0 keeps it sparse). | 
| `CELLD_RECOVERY_RETRY_MS`/`CELLD_RECOVERY_RETRIES` | Node-log recovery retry pacing (defaults 1,000 and 240); recovery reads bundles in 512 MiB windows and checkpoints every 32 cell epochs. | 
| `CELLD_WAKER_TICK_MS` | Interval of the fleet waker's alarm cleanup pass (default 60,000). | 
| `CELLD_ALARM_RESIDENT_MS` | How close to its next alarm a cell (a Workflow instance, say) stays resident rather than hibernating (default one hour). | 
| `CELLD_DOCKER`/`CELLD_CONTAINER_PLATFORM`/`CELLD_CONTAINER_RUNTIME` | Containers: the container CLI used to build and pull (default `docker`; Podman works), the deploy build platform (default`linux/amd64`), and a node-wide OCI runtime such as`runsc`or`kata`. | 
| `CELLD_OTEL` | `0`off,`1`Parquet to the bucket, or an OTLP/HTTP collector base URL (see Chapter 8). | 
| Rejected at startup | `CELLD_OUTPUT_GATE`(the gate is always on; celld always waits for the configured durability proof),`CELLD_STORAGE_PROBE`,`CELLD_SHUTDOWN_DRAIN_MS`,`CELLD_DRAIN_TOKEN_WAIT_MS`,`CELLD_WORKER_LOADER`,`CELLD_MAX_LOADED_WORKERS`,`CELLD_OTEL_SINK`,`CELLD_AI_BINDING`,`CELLD_AI_URL`,`CELLD_REBALANCE_BATCH_CELLS`, and a handful of older tuning knobs. A node with any of them set does not start. | 

This chapter draws on: celld: documentation at v0.6.0 (9 entries) · celld: release notes (5 entries) · Cloudflare documentation (4 entries). The full entries are in the Bibliography.


---

<!-- https://tlockney.github.io/celld-book/compatibility.html -->

celld runs the Workers runtime (module Workers, `fetch`, JS RPC, service bindings, Durable Objects, static assets, KV, Queues, Workflows, R2, Dynamic Workers, facets, and containers) with Durable Objects as the stateful core. The scope rule again: **a configuration or binding that is not available must fail loudly, at deploy or first use; a silent gap is a bug.** Each service has its own page under `docs/services/`, with a how-it-works narrative, a worked example, and a "differences from Cloudflare" list, while the runtime-API sections stay on the compatibility page. The pages list *only* an unavailable feature, a celld-specific limit, or an observable difference: "if an entry has no note, celld intends to match the linked Cloudflare API". Each surface is graded *Yes* (implemented, except the listed differences), *Partial* (a substantial part unavailable), *Experimental* (can change without notice), or *No*. On that scale every service celld carries is *Yes* except Containers (*Experimental*). In the runtime table only Node.js compatibility and the Cache API are *Partial*, and BroadcastChannel is the lone *No*.

## Runtime API surface: the parts that matter

| Surface | Status on celld | 
|---|---|
| Fetch / Request / Response / Headers | Yes. The `cache`request option is unavailable; celld removes`Content-Length`from a Worker response (preserves it for`HEAD`);`Headers`accepts values above U+00FF and decodes response values as UTF-8 | 
| Context ( `ctx`) | Yes. `passThroughOnException()`is a no-op;`ctx.facets`is available only inside a Durable Object;`ctx.exports`holds`default`and each entrypoint, and`fetch()`on its stubs sends an HTTP request | 
| Main module | As in workerd, every export of the main module must be a handler object or a class; `export const X = "..."`makes the Worker fail to start | 
| Handlers | Yes. `fetch`,`alarm`,`scheduled`,`queue`, WebSocket handlers, RPC;`tail`and`email`unavailable | 
| JS RPC | Yes. A stub cannot cross an isolate boundary; an `AbortSignal`passes through on the same node but not across a node boundary; retries only when the peer attempt provably did not start | 
| Streams / Encoding | Yes. An HTTP stream expires after 60 s unclaimed or inactive (renewed by successful reads), and an expired or unknown stream errors rather than reporting EOF | 
| WebSockets | Yes. Inbound (hibernatable, with attachments and tags) and outbound; a 1 MiB per-isolate input budget per non-terminal frame; a transport cannot move to a new owner, so reconnect with a stable operation ID; a mid-frame connection failure closes with 1012; the output gate holds each frame only for its own proof, so a `webSocketMessage()`handler can stream frames while it runs; a close is`wasClean: true`whenever the peer sent a close frame | 
| Web Crypto | Yes, including `wrapKey`/`unwrapKey`, RSA signing, HKDF/PBKDF2 derivation, and Ed25519 (also spelled`NODE-ED25519`) and X25519 with`raw`import and export of the 32-byte point. Remaining limits are algorithm-specific: ECDSA P-256 with SHA-256 only, AES-GCM tags 96–128 bits, a secret key cannot use`jwk`with`exportKey()`/`wrapKey()`, and X25519 rejects a low-order peer key | 
| `node:`imports | Partial. assert, async_hooks, buffer, diagnostics_channel, events, path, stream, timers/promises, util, os; crypto/zlib partial; `node:fs`covers`access`/`mkdir`/`realpath`/`stat`/`readFile`over a per-request`/tmp`and a read-only`/bundle`; http, net, tls, dns import but throw on first call; the bundler honors synchronous CommonJS`require()`of built-ins | 
| D1 | Yes. A cell; one writer; results capped at 100,000 rows / 32 MiB; a `TEXT`value that is not valid UTF-8 decodes with U+FFFD, as in workerd (the stored bytes do not change) | 
| KV · Queues · Workflows · R2 | Yes (remaining differences below) | 
| HTMLRewriter · TCP sockets · EventSource · MessageChannel | Yes. A TCP socket cannot outlive its event (a Durable Object reconnects next event); TLS is verified against a bundled Mozilla root store; celld does not block the ports Cloudflare blocks, so the fleet network controls egress | 
| Cache | Partial. An always-miss cache: `put()`validates and consumes the response but stores nothing,`match()`returns`undefined`,`delete()`returns`false` | 
| BroadcastChannel | No. The class is defined so a bundle loads, but its constructor throws rather than acting as a silent stub | 

## Dynamic Workers, facets, and containers

**Dynamic Workers** is the Worker Loader under its current name, graded *Yes*. A deployment declares its loaders in `wrangler.jsonc`: `"worker_loaders": [{ "binding": "LOADER" }]`. A loaded Worker can receive Service Binding capabilities through `WorkerCode.env` alongside structured-clone values (1 MiB total), so a parent can hand a child a `ctx.exports` entrypoint to call back on; `getEntrypoint()` and `getDurableObjectClass()` take only `props`. The process holds at most 256 live Dynamic Workers, 255 per script generation, and that limit is not tunable; module sources total at most 64 MiB. `globalOutbound` cannot `connect()` or open a WebSocket. `WorkerCode.limits` (and the `limits` option of `getEntrypoint()`) *enforces* `cpuMs` and `subRequests`. A `WorkerCode.tails` array of Service Binding Fetchers receives one invocation report per finished fetch (request metadata, response status, up to 256 KiB of console records, the uncaught exception, and the outcome), delivered after the response, with a Tail failure logged rather than surfaced. `allowExperimental` is rejected. The Wrangler `worker_loaders` entry itself accepts only `binding`; `limits` and `tails` belong in the `WorkerCode` object, and the deploy stops if they appear in the config entry. Three rules match workerd exactly and can break code written loosely against an earlier release: `WorkerCode` **requires** `compatibilityDate`, a wasm entry in `modules` must be `{ wasm: bytes }` (bare bytes are refused), and a relative import inside a module subdirectory resolves from the importing module's name.

**Durable Object facets**: `ctx.facets.get()`/`abort()`/`delete()` attach a child object with its own SQLite database. The class comes from a Worker Loader binding (`worker.getDurableObjectClass()`) or from `ctx.exports` for a `DurableObject` class the Worker exports *without* a storage migration; that facet runs in the root's isolate. A Durable Object binding cannot supply one. Each facet lives in **its own SQLite file with its own replication stream** under the root cell's bucket prefix, sharing the root's ownership record and epoch. The consequence: a facet write commits in the facet's own database, so rolling back a root transaction does not undo a facet call inside it. An application that needs one atomic commit must keep that state in one database. celld holds a facet's outbound effects and replies until the facet's own stream proves the call's writes, and a move proves every facet stream before the new owner opens the root. `clone()` is unavailable. Two more limits: a facet cannot set an alarm (`storage.setAlarm()` throws inside one, so the root holds the schedule), and facets nest to a total depth of four counting the root, with names up to 256 bytes. The `examples/facets` project shows the whole pattern, including the callback capability.

**Containers** are *Experimental*: "the configuration keys, the `ctx.container` surface, the node-side defaults, and the security boundary can change without notice". A container makes a Durable Object the supervisor of one container, driven by `@cloudflare/containers` as published; the **Sandbox SDK** (`@cloudflare/sandbox`, on the `cloudflare/sandbox` image) runs on top unchanged. A `containers` entry accepts exactly `class_name`, `image`, `name` (accepted, unused), `instance_type`, `max_instances`, and the celld-only `runtime` override; the class must be a SQLite-backed Durable Object of the same script. `celld deploy` builds or pulls the image with the Docker or Podman CLI (`CELLD_DOCKER`), for `linux/amd64` by default (`CELLD_CONTAINER_PLATFORM`), saves it once to the bucket under `deploy/images/`, and every node loads it on first use; each fleet node needs a container engine socket. `instance_type` maps to Cloudflare's CPU and memory tiers: `lite` (alias `dev`, the default: a sixteenth of a CPU and 256 MiB), `basic`, `standard-1` through `standard-4`. Disk size is not enforced. `max_instances` caps a class fleet-wide through the shared node sample, so it can overshoot by one refresh cycle. The node fences container bridges with nftables before the first start: `enableInternet: true` reaches only the Internet, never the node, its peers, or private ranges, and `false` is an internal bridge with no route out. Container disk is ephemeral; it dies with a move, restart, or reset. Idle eviction respects `setInactivityTimeout()` (default 10 minutes). Implemented: `running`, `start()`, `monitor()`, `destroy()`, `signal()`, `getTcpPort()`, `exec()`, `setInactivityTimeout()`; not implemented: `inspect()`, snapshots, and outbound interception. A sandbox that moves nodes loses its container, so the first call after a move can throw the SDK's `OperationInterruptedError`, matching Cloudflare's own restart behavior.

## KV

- **No edge cache.**- `cacheTtl`has no effect and- `cacheStatus`is- `null`. KV is a durable store, not a CDN.
- A value above 1 MiB requires the fleet bucket (small values are in-cell). Large values are stored under their ownership epoch with an epoch-qualified row reference, so an old owner cannot delete the current value.
- **One writer per namespace.**Add namespaces, not writers. A namespace ID accepts the Cloudflare hex form or any stable string.
- Operate with `celld kv get/put/delete/list`and`bulk`variants (Wrangler file format, so it interops with`wrangler kv bulk`).

## Queues

- **One writer per queue**; scale with more queues. Producer calls share transactions and durability rounds, each message gets a time-ordered ID, and a broker admits up to 256 concurrent producer calls (committing at most 64 per transaction, four proofs overlapping), refusing more with an error the producer can retry. One queue sustained 7,357 sends per second over a 300,000-send soak with exact delivery. The refusal surfaces as- `cell overload: admission refused`in a caught producer error, so a Worker can relay the 503.
- **One consumer script per queue.**A deployment where two scripts consume one queue fails. The consumer script may also export- `fetch()`:- `examples/queues`exports both- `fetch`and- `queue`from one script. Lab 2, Part 5 shows what happens when a declared consumer's script loses its- `queue()`handler: the local node exits with- `queue consumer has no queue handler`. Consumer settings follow Cloudflare:- `max_batch_size`10 (max 100),- `max_batch_timeout`5 s (max 60),- `max_retries`3,- `max_concurrency`up to 250,- `retry_delay`,- `dead_letter_queue`an ordinary queue; celld validates each bound at deploy time, so a bad value fails the deployment rather than the first delivery. A retried message becomes visible again after- `delaySeconds`from- `retry()`or- `retryAll()`, defaulting to the consumer's- `retry_delay`; celld adds no exponential backoff, so an application that wants one computes the delay from- `message.attempts`. When- `attempts`passes- `max_retries`, celld moves the message to the- `dead_letter_queue`, or- **deletes it**if the consumer names none. Limits: messages up to 128,000 bytes, 100 per- `sendBatch()`(256,000 bytes in total),- `delaySeconds`up to 86,400.
- **Messages.**The- `contentType`option selects- `"v8"`,- `"json"`,- `"text"`, or- `"bytes"`, and the- `queues_json_messages`compatibility flag chooses the default, as on Cloudflare. A producer entry's- `delivery_delay`sets the default delay for every message sent through that binding.
- Messages are retained four days, **not configurable**. Pull consumers, the Queues HTTP API, dashboard controls, manual consumer attachment, R2 event notifications, and Queue event subscriptions are not available.
- Operate with `celld queue info/peek/purge/pause/resume/redrive`.

## Workflows

- **Replay is the discipline.**A running workflow is stored as steps. After a crash,- `run()`replays from the start, so code- *outside*a step runs again, and a crash after a step side effect can run that step's callback again. The Workflows page does not list this as a celld difference, because it is how Cloudflare Workflows work too. Everything meaningful goes inside a step, and steps must be idempotent. Lab 3, Part 3 counts it: across one durable sleep, the top of- `run()`ran twice while each step ran once.
- **Retention.**A successful or failed instance is kept 30 days by default, each duration in the- `retention`option can be at most 30 days, and completed runs can be deleted manually.- `locationHint`accepts Cloudflare's values but fleet ownership picks the actual location.
- **Limits.**Non-step work cannot stay pending more than 60 seconds; a step result, event payload, and workflow parameters are each capped at 1 MiB. Rollback, sensitive step results, and- `ReadableStream`step results are unavailable. A- `workflows`entry cannot carry- `schedules`,- `limits`, or a- `script_name`naming another script. The Workflows REST API and- `wrangler workflows`do not operate against celld.
- **Defaults.**- `step.do()`retries 5 times, 10-second delay, exponential backoff, 10 minutes per attempt, stoppable with- `NonRetryableError`;- `waitForEvent()`times out after 24 hours; an instance within an hour of its next alarm stays resident (- `CELLD_ALARM_RESIDENT_MS`).
- **Create-once semantics: verify.**The page says nothing about- `create()`with a terminal instance's ID, and nothing about- `pause()`/- `resume()`/- `restart()`. Cloudflare refuses the duplicate ID; celld replaced it in v0.4.0 (see Appendix C). By the page's own rule the silence means celld intends to match Cloudflare, but no release note calls it out as a fix. Verify against your installed release if you depend on create-once semantics.

## R2

- The R2 binding uses your **fleet bucket**under`r2/<bucket_name>/`, which is how the fleet bucket earns its name. An object's`version`equals its content ETag, so identical content produces the same version. The version comes from the object store: most stores report no version identifier, so the ETag becomes the version.`celld dev`'s local store instead numbers each write from a store-wide counter, so identical bytes under two keys get different versions (Lab 2, Part 4 shows it). Either way, an application must not use a version to count writes, and should not rely on version equality for de-duplication without checking the store it deploys to;`checksums.md5`is the content hash on both. The five content headers are stored as object headers;`customMetadata`,`cacheExpiry`, checksums, and storage class travel together in one JSON value under the`celld-r2`user-metadata name (`celld_r2`on Azure, which refuses a hyphen; #209).`celld r2 get|head|put|delete|list`operates these objects without a running node.
- **Access is through the binding only.**There is no public bucket URL, no presigned URL, and no S3 endpoint into an R2 binding, so an application must put a Worker in front of any bytes it wants to publish.
- **Interop.**An object another tool wrote still reads through the binding: its user metadata becomes its- `customMetadata`and its headers become its- `httpMetadata`. Use- `celld r2 put`to write the complete record.- `delete(keys)`removes up to 1,000 keys in one call.
- `ssecKey`and- `jurisdiction`are not available. A conditional write cannot use a streamed body larger than 8 MiB.
- Multipart: `createMultipartUpload()`accepts no checksum; a multipart upload**cannot resume on another node or after a restart**; celld cannot replace a part the store already holds; out-of-order parts are limited to 256 MiB of memory, and completion cannot change the stored part order.
- **Keys.**celld keeps empty key segments, so- `a/b`,- `/a/b`,- `a//b`, and- `a/b/`are four objects, as in Cloudflare R2. The store percent-encodes keys with non-ASCII or special characters (- `přehled.html`becomes- `p%C5%99ehled.html`) and- `list()`decodes them, so a listed key is the key- `put()`received. But- `list()`sorts and compares- `startAfter`by the- *encoded*form, so a key with one of those characters can land in a different position than on Cloudflare, and- `startAfter`can skip or include a key Cloudflare would not. A celld- `cursor`has no such problem. An object v0.5.1 wrote as- `photos/`stays at- `photos`.

## Wrangler configuration

`celld deploy` reads `wrangler.jsonc` or `wrangler.json`, **not** `wrangler.toml`. Supported keys: `$schema`, `name`, `main`, `no_bundle`, `compatibility_date`, `compatibility_flags`, `durable_objects`, `migrations`, `assets`, `services`, `triggers`, `vars`, `d1_databases`, `kv_namespaces`, `queues`, `workflows`, `r2_buckets`, `worker_loaders`, `containers`, `define`, and `rules`. `define` and `rules` are both handed to the esbuild run, so neither combines with `no_bundle`; a rule's `type` is `Text`, `Data`, or `CompiledWasm` with globs of the form `**/*.ext`, and a rule that gives `**/*.wasm` any type but `CompiledWasm` stops the deploy. The `name` must be 1–63 lowercase ASCII letters, digits, or internal hyphens. Anything else (`routes`, unknown keys) stops the deploy with an error naming the key. Compatibility flags (`js_rpc`, `sqlite_vec`, `websocket_standard_binary_type`, `delete_all_deletes_alarm`, `fetcher_no_get_put_delete`, and the assets navigation flags) are honored; unmodeled flags are accepted without effect.

```
{
  "$schema": "./node_modules/wrangler/config-schema.json",
  "name": "chat",
  "main": "src/index.ts",
  "compatibility_date": "2026-01-01",
  "durable_objects": {
    "bindings": [
      { "name": "ROOMS", "class_name": "ChatRoom" }
    ]
  },
  "migrations": [
    { "tag": "v1", "new_sqlite_classes": ["ChatRoom"] }
  ],
  "d1_databases": [
    { "binding": "DB", "database_name": "ledger" }
  ],
  "kv_namespaces": [
    { "binding": "SESSIONS", "id": "sessions-prod" }
  ],
  "queues": {
    "producers": [ { "binding": "OUTBOX", "queue": "outbox" } ],
    "consumers": [ { "queue": "outbox", "max_batch_size": 32 } ]
  },
  "r2_buckets": [
    { "binding": "FILES", "bucket_name": "files" }
  ],
  "triggers": { "crons": ["*/5 * * * *"] },
  "worker_loaders": [ { "binding": "LOADER" } ],
  "containers": [
    { "class_name": "Sandbox", "image": "./Dockerfile",
      "instance_type": "basic", "max_instances": 4 }
  ]
}
```
## D1 is a cell

A D1 database is just a cell holding one SQLite database, replicated to the fleet bucket, so it inherits the fencing, replication, and durable acknowledgement of any Durable Object. **One database has one writer**; a fleet gets more capacity from more databases, never from a larger one. Migrations are `NNNN_description.sql` files in `migrations/` (the extension is case-insensitive, and a custom `migrations_dir` must be a relative path inside the project), applied in numeric order exactly as Wrangler does, in one transaction per file. The `celld d1` command runs SQL and migrations against a deployed database, signed with the fleet secret. Import from Cloudflare with `wrangler d1 export` then `celld d1 execute DATABASE --file export.sql`; a migration already applied does not run twice when the history arrives with the data. `dump()`, Time Travel, the D1 REST API, and the `wrangler d1` commands do not operate against celld. A SQLite `TEXT` value that is not valid UTF-8 decodes with U+FFFD, as workerd decodes it; store arbitrary bytes in `BLOB`.

## Cron, alarms, and WebAssembly

Cron triggers run the `scheduled` handler on celld's own durable alarms, one minute resolution in UTC, exactly once per occurrence fleet-wide. One handler runs at a time per script; a handler can run late but never early. A thrown handler is retried with backoff (starting at 4 seconds, doubling, abandoned after 6 failures, and only the expression that threw), and `controller.noRetry()` cancels the retry. Note the day-of-week convention: 1 is Sunday, the same as Cloudflare and opposite to most cron dialects. A service-binding target cannot run its own cron triggers.

Wasm imports give the compiled module (Wrangler's rule), uploaded beside the bundle and marked with the `wasm-v1` feature so a mixed fleet fails at deploy time. celld compiles each module once per process and reuses it across isolates; `worker-build` (workers-rs) produces a shim that is a normal `celld deploy` entry point. Prebuilt deployments work the same way: with `no_bundle: true` the entry JavaScript ships byte-for-byte and celld applies Wrangler's default `**/*.wasm` patterns below the entry's directory, so `main: "./dist/shim.mjs"` importing `"./add.wasm"` just works, without esbuild on the node.

This chapter draws on: celld: documentation at v0.6.0 (9 entries) · celld: release notes (5 entries) · Cloudflare documentation (4 entries). The full entries are in the Bibliography.


---

<!-- https://tlockney.github.io/celld-book/designing.html -->

The model pays off when you **divide the application into named, stateful units from the start**: one cell per user, per room, per agent, per document, per device. Because cells share no database, the classic distributed-system problems (locking, message buses, hot shards in a shared table) simply do not arise; the cell *is* the partition.

## The workloads the model fits

- **Real-time applications.**A multiplayer game, chat room, or collaborative document is one cell holding both the WebSocket connections and the room's state, with no lock and no external message bus. A single Durable Object can coordinate thousands of clients through the hibernation API.
- **Agents.**Each AI agent is one cell holding memory, schedule, and inbox in its own SQLite; an idle agent hibernates to the bucket, so a large agent fleet costs almost nothing between events.
- **Sharded web applications.**One cell per user or tenant shards the app from the start; the contention of one shared database never appears because no shared database exists. KV, Queues, D1, and R2 round out the toolbox for the parts that are not per-entity state.

## Entities, not processes

A cell models an **entity**: a named unit with state that persists indefinitely, such as a concert, a user, or a document. A durable-execution engine (Temporal, Restate, Azure Durable Functions) models a **process**: a sequence of steps that ends, such as an order pipeline. celld's Workflows implementation is the process primitive built on the entity primitive, and its replay semantics (see Chapter 6) are exactly the tradeoff that primitive carries: steps must be idempotent, because a crash re-runs them. Pick the shape that matches the problem rather than the tool you have.

## A working cell

This is the whole application shape, a hibernating chat room with durable state:

```
import { DurableObject } from "cloudflare:workers";
export class ChatRoom extends DurableObject {
  async fetch(req: Request) {
    const pair = new WebSocketPair();
    const [client, server] = Object.values(pair);
    this.ctx.acceptWebSocket(server);          // hibernatable — the room can sleep
    return new Response(null, { status: 101, webSocket: client });
  }
  async webSocketMessage(ws: WebSocket, message: string | ArrayBuffer) {
    for (const peer of this.ctx.getWebSockets()) {
      if (peer !== ws) peer.send(message);      // one room, no message bus
    }
  }
  async alarm() {
    await this.ctx.storage.put("lastAlarm", Date.now());
    this.ctx.storage.setAlarm(Date.now() + 60_000);  // durable self-schedule
  }
}
```
## Operational rules of thumb

- **Batch WebSocket messages.**Each frame costs a context switch; pack many small logical messages into one frame with an envelope format. Fewer, larger messages beat many small ones.
- **Keep the constructor cheap.**It runs on every wake, including each message to a hibernated cell. A schema-version check inside- `blockConcurrencyWhile()`belongs there; loading the cell's state does not, so restore from- `storage`in the handler.
- **Route a cell's traffic to its owner node when latency matters.**The versioned peer tunnel lets any node ingress any cell, but the warm path (zero bucket operations, p50 ≈ 1.1 ms) only exists when the request lands on the owner.
- **Outbound WebSocket connections do not survive a move.**An outbound DO socket keeps the cell resident and dies with the node/owner move; keep connection intent in storage and reconnect after activation. A WS transport cannot move to a new owner, so reconnect with a stable operation ID.
- **Make remote operations idempotent.**celld does not retry a proxied call after transmission starts. Use a stable operation ID and design handlers to tolerate a retry.
- **One writer per cell, per D1 database, per KV namespace, per queue.**Shard by adding entities, never by growing one.

## Cloudflare's rules, on celld

Cloudflare's Rules of Durable Objects is the standard design checklist, and most of it carries over unchanged. The table sets each rule against celld's Durable Objects page. Three rows need the most attention: renames do not carry over, the constructor needs a precise rule, and celld has more ways to stop an object than Cloudflare does.

| Cloudflare's rule | On celld | 
|---|---|
| Model one object per "atom" of coordination; never route all traffic through a global singleton | The same, and a singleton costs more here: every call to a cell is forwarded to the one node that owns it, so one hot cell loads one machine of your fleet. | 
| Use deterministic IDs ( `getByName()`,`idFromName()`) | Both work. The id is an HMAC-SHA-256 of the name under a key derived from the script name and the class name, so one name reaches one cell from any node. A `newUniqueId()`id cannot be derived again, so keep its string form. | 
| Rename or delete a class through migrations | Does not carry over.A`migrations`entry accepts only`tag`and`new_sqlite_classes`; a class rename, delete, or transfer stops the deployment. Renaming the Workerscriptchanges every derived id: the renamed script reaches new, empty cells while the old data stays under the old ids. Keep script and class names stable, or migrate the data first. | 
| Give a location hint | Accepted with Cloudflare's values, but fleet ownership decides where a cell runs. celld makes no placement, migration, or jurisdiction promise, and the jurisdiction calls throw. | 
| Run schema migrations in the constructor, inside `blockConcurrencyWhile()`; use it for nothing else | Yes, but keep it to a cheap schema-version check: the constructor runs on every wake, and a `blockConcurrencyWhile()`or transaction longer than 30 seconds resets the object and rolls the transaction back. | 
| Treat in-memory state as a cache; persist what matters | Stronger here. Idle eviction drops memory, and ownership moves when a node drains, when rebalancing moves a hibernated cell, and when a node is lost. Only storage and WebSocket attachments survive. | 
| Design for unexpected shutdowns: write progress as you go | celld has more ways to stop an object: idle eviction, a drain handoff that cancels firing alarms and active internal fetch/RPC handlers (Chapter 5), a rebalancing move, a self-fence that exits with code 3 (Chapter 4), and SIGKILL when the orchestrator's stop grace is too short. Persist each step before the next await that could be the last. | 
| Rely on the output gate; don't `await`writes for safety | The same promise, with a stronger proof: a response waits until a bucket or fleet proof covers every write it can reveal, and a WebSocket frame waits only for its own proof. `transaction()`and`transactionSync()`group writes; a nested transaction that fails discards only its own writes. | 
| Guard against races across non-storage I/O | The same interleaving rule: storage calls are synchronous and never interleave, but an outbound `fetch()`or RPC`await`lets another event run. Keep a read-modify-write free of such awaits, or re-check a version after the await before writing. | 
| Make alarm handlers idempotent | Required, and celld helps: `alarm()`receives`retryCount`and`isRetry`, and a drain arms a durable wake so the successor runs a cancelled alarm at least once. | 
| Use hibernatable WebSockets and `serializeAttachment()` | Supported, with attachments and tags. A hibernatable socket survives hibernation on the same node but closes with code 1012 when the cell moves to a new owner, so clients must reconnect. | 
| An object doesn't know its name, so store it in an `init()`call | Mostly unnecessary: `ctx.id.name`carries the name for names up to 1,024 UTF-8 bytes, which is Cloudflare's own limit. A`newUniqueId()`cell has no name and still needs the record. | 
| Clear an object with `deleteAll()` | Available. celld's docs do not say whether it also clears a scheduled alarm, so call `deleteAlarm()`as well. | 
| Plan for roughly 500–1,000 requests per second per object | That is Cloudflare's measurement. celld publishes no per-cell figure; a warm request on the owner does zero bucket operations, but every write waits for its durability proof. Measure on your own fleet and storage. | 
| Test with `@cloudflare/vitest-plugin` | That pool runs on workerd. celld's differential conformance tests hold celld to workerd's output, but celld's docs name no celld-backed test pool, so test ownership, durability, and moves against `celld dev`or a fleet. | 
| Prefer RPC methods, and always `await`them | The same. An RPC stub cannot cross an isolate boundary, an `AbortSignal`does not cross a node boundary, and a proxied call is not retried once transmission starts, so keep operations idempotent. | 

This chapter draws on: celld: documentation at v0.6.0 (9 entries) · celld: release notes (5 entries) · Cloudflare documentation (4 entries). The full entries are in the Bibliography.


---

<!-- https://tlockney.github.io/celld-book/verdict.html -->

## Reliability, testing, and telemetry

celld stakes three promises, and its testing is organized around breaking them: *an acknowledged write is durable*, *a cell has one writer at a time*, and *code written for Cloudflare behaves the same on celld*. The engineering is unusually rigorous for a pre-1.0 project, and worth respecting on its merits.

### Four test layers

- **Differential conformance.**Each program runs twice, once on workerd and once on celld, identical bytes, and the outputs must be equal. A test cannot agree with celld's own runtime by accident.
- **Exhaustive specification.**The coordination protocol is specified in TLA+ and model-checked at small configuration, with pinned expected verdicts, most of them failures that model bugs the protocol once had, kept as a canary. The model found four bugs and a split-brain that lost an acknowledged write, none of which had surfaced in review or testing. The fencing argument itself is checked, not asserted.
- **Deterministic simulation.**The coordination logic is a pure decision core with no I/O; a seeded scheduler injects latency, CAS races, lost responses, drifting clocks, and crashes at every await point. Safety and liveness properties must survive tens of thousands of seeds; the core protocols have run through millions of schedules. They also test the checkers: deliberately broken protocol variants must be caught.
- **Live fleet lab.**Real VMs, a real bucket, fault injection between verification passes (SIGKILL mid-write and delete the local DB; freeze an owner and unfreeze it; cut a node off from the bucket; throttle the bucket to 429s; stop a full host). Every scenario's verification sweep found zero lost acknowledged writes.

### Numbers with their conditions

| Measurement | Result | 
|---|---|
| Epoch fence under contention | 500 claimants, 5,500 attempts, one writer per epoch, zero violations | 
| Warm resident request | zero bucket operations; p50 ≈ 1.1 ms, p99 ≈ 7 ms (fixed host) | 
| Durable write | one bucket round trip with bucket proof; a fleet proof (≥2 nodes) answers on follower fsync. v0.3.0 measured 10× lower write latency and 100× fewer Class A S3 ops | 
| Concurrent writes to one cell | join a single shared upload, so throughput is not one round trip per write | 
| Scale (measured) | 10 nodes × 4 vCPU / 8 GB held 10,000 resident cells + 20,000 concurrent WebSockets | 
| Node failure | stopping 2 of 10 nodes: every cell's data available again in ~11 s at the tail | 
| Queue throughput | one queue sustained 7,357 sends/s over a 300,000-send soak with exact delivery (up from a few hundred before the v0.4.1 producer rebuild) | 
| Whole-fleet restart | nodes that restart together recover acknowledged writes: a restarting node serves follower fragments while recovering its predecessor, with checkpoints and a heartbeat so a retry skips finished work | 

### Telemetry

Telemetry is off by default and costs nothing until `CELLD_OTEL` is set; that one variable also picks the sink. `CELLD_OTEL=1` writes **Parquet** files to the fleet bucket under the `telemetry/` prefix, partitioned by node and hour, so a fleet with a bucket has observability with no other service; DuckDB queries the files directly. `CELLD_OTEL=https://collector.internal:4318` (a full HTTP(S) base URL) sends the same data as OTLP/HTTP protobuf instead, with `/v1/traces` and `/v1/logs` appended; celld does not read `OTEL_EXPORTER_OTLP_ENDPOINT`. The OTLP exporter retries a batch up to five times with jittered backoff (honoring `Retry-After`, 30 s cap) on 408/429/502/503/504, drops it on a permanent refusal, and bounds its buffer at 8,192 events so a collector outage cannot grow memory; new telemetry is dropped and counted rather than blocking requests. Spans cover each request, cell event (fetch, alarm, RPC, WebSocket message), outbound `fetch()`, and cell start, plus every `console.log` as a log record joined to its trace. W3C `traceparent` is read and emitted (a malformed one starts a new trace), so traces join the systems in front of and behind celld.

```
CREATE VIEW traces AS SELECT * FROM
  read_parquet('s3://YOUR-BUCKET/telemetry/traces/*/*/*/*/*/*.parquet');
SELECT name, duration_us, trace_id FROM traces
  ORDER BY duration_us DESC LIMIT 20;
```
Defaults: 5-minute / 5 MB flush (the byte threshold is an estimate, so a batch can slightly overshoot it), 30-day retention swept at startup and every six hours (`none` hands lifecycle to your own rules), `OTEL_TRACES_SAMPLER` for fractional sampling with a consistent per-trace decision across nodes. For a near-live view, set `CELLD_OTEL_FLUSH_MS=5000` and run the one-hour compaction job on a maintenance node; a short flush without compaction makes DuckDB open thousands of small files. There are no metrics yet. That is a named gap, and the span durations cover most of what a metric would answer.

## Security boundaries

celld is a beta, and it says so plainly: **not safe for hostile multi-tenant use**, and security fixes apply to the latest release only. The threat model is single-tenant with a trusted operator. Within that, the boundaries are explicit.

- **Two listeners.**The public Worker listener (- `--listen`) is the only thing a load balancer or firewall should expose; it reserves- `/.well-known/celld/health`and hands every other path to the Worker. The internal listener (- `--internal-listen`) carries the peer protocol and the operator API and must stay on a private network or encrypted overlay. Most of the operator API is unauthenticated: anyone who can reach it can inspect state, evict cells, or stop the process. The one exception is the D1 route, which authenticates with the fleet secret because it runs caller-supplied SQL.
- **The bucket is the root of authority.**It holds deployments, cell state, ownership records, node leases, the shared peer-authentication secret, large KV values, and R2 objects. Scope credentials to one bucket, use a prefix to share a bucket safely, and rotate on any suspicion of disclosure.
- **One writer per cell.**The epoch fences each cell; a node that loses its lease cannot modify current cell state. Peer requests authenticate with HMAC, body signature, clock limit, and replay protection, but the peer tunnel carries plain HTTP for cell fetch/RPC, so- **the private network or encrypted overlay is the confidentiality boundary**, not the protocol.
- **No TLS termination.**Put public TLS in your ingress proxy and use WireGuard/Tailscale for the internal plane. By default celld ignores- `X-Forwarded-Host`/- `X-Forwarded-Proto`; set- `--trust-forwarded-headers`only behind a trusted proxy that rewrites both, and it reads the last value so a direct client can't spoof it.
- **Application auth is yours.**celld does not authenticate your application's users and does not enforce per-cell quotas; a defective cell can only touch its own database, but it can consume resources on its fleet node. Enforce request limits yourself via- `CELLD_MAX_REQUEST_BODY_BYTES`and- `CELLD_MAX_CELL_REQUESTS`.
- **Egress is your network's job.**Outbound TCP (- `cloudflare:sockets`) verifies TLS against a bundled Mozilla root store, but celld does not block the ports Cloudflare blocks. Containers get an nftables-fenced bridge (- `enableInternet: true`reaches the Internet only, never the node, its peers, or private ranges), but that fence is experimental by the docs' own label, and macOS- `celld dev`keeps container egress on with a warning. Internal host functions are not exposed to application code.

## Tradeoffs and a verdict

| Cloudflare Durable Objects | celld | |
|---|---|---|
| Placement | vendor scheduler, opaque | your fleet; ownership = a lease in your bucket | 
| State & durability | platform DO storage | SQLite replicated as LTX to your bucket, RPO=0 (bucket or fleet proof) | 
| Failure domain | shared platform (tenant-coupled) | your nodes + your bucket provider | 
| Data services | KV, Queues, D1, R2, Workflows on the platform | all five graded Yes, backed by your bucket / cells | 
| Containers & sandboxes | Cloudflare Containers, Sandbox SDK | Experimental: a Docker/Podman engine per node, images in your bucket, an nftables-fenced bridge | 
| Placement balance | platform scheduler | weight-proportional balancing of hibernated cells, pausable fleet-wide | 
| Multi-tenancy | platform | not yet; one application per fleet | 
| Deployments | instant, platform-managed | adopt in place without restart; module digests verified; one app per fleet | 
| Observability | Cloudflare dashboard | Parquet in your bucket + DuckDB, or OTLP | 
| Write latency | platform-managed | one bucket round trip (single node) or follower fsync (fleet) | 
| Ingress / TLS | platform | your proxy; peers over Tailscale/WireGuard | 
| Correctness claims | platform SLA | explicit + tested: TLA+, simulation, differential conformance | 

What celld buys you is **placement and blast radius you choose**: no shared Durable Objects scheduler can couple your application to another customer's workload, and when a cell misbehaves the evidence is on your disk (ownership records, SQLite and LTX files, logs), answerable with `sqlite3` and `grep` rather than a status page. It buys RPO=0 as a hard guarantee, which most self-hosted setups cannot claim, and it buys portability: the same Worker/DO code runs on workerd or on your fleet. KV, Queues, Workflows, and R2 close most of the "different primitive" gap; facets, HTMLRewriter, TCP sockets, and full Web Crypto close most of the runtime-API gap; and containers and sandboxes supervised by a Durable Object, on your own nodes, open a gap in celld's favor. The toolbox for building a full application on cells exists, not just the entity primitive.

What it costs is operational ownership. Balancing moves only hibernated cells and counts them by node weight. Updates are manual behind a `current` pointer. Cold restores touch object storage (paged, for a large cell). A durable write costs at least a follower fsync and eventually a bucket upload. The store must be on the qualified list: a wrong store fails the startup probe, which cannot be disabled, and a store that silently ignores conditional writes or ranged reads fails late and dangerously. The compatibility surface is still a strict subset of Cloudflare's. The platform services (AI, Vectorize, Hyperdrive, Browser Rendering, Email, Python) are absent, the Cache API is an always-miss, and KV/Queues/Workflows/R2 keep real gaps (no edge cache, 4-day fixed queue retention, multipart that cannot resume). Containers are experimental with a self-declared movable security boundary. The beta label changes none of the caveats: one application per fleet, no Windows, an operator API that can change between releases, full-stop upgrades (v0.4.1→v0.5.0 with no binary rollback, and v0.5.1→v0.6.0 under fleet durability), and WS transports that cannot move between owners. One semantic changed under running applications in v0.6.0 and deserves a check in any code that relies on it: a facet write does not commit atomically with the root object's transaction.

The verdict for practical use: the model is the best primitive in distributed systems right now, and celld's engineering discipline (TLA+, seeded simulation, differential conformance, live fault injection) is more rigorous than many production platforms. celld is a credible self-hosted platform for stateful serverless: the data-service toolbox exists, the fleet balances itself, large cells restore page by page, a cell can supervise a container, and celld itself has stopped calling it an alpha. It is ready for a team that wants DO semantics on its own infrastructure and can own the ops: agent fleets (with sandboxes), per-tenant sharding, real-time rooms, self-hosted stateful services. It is still not for hostile multi-tenancy, zero-ops, or anything that needs the full managed platform surface. Treat it as an unusually well-proven beta: pin releases, respect the upgrade cliffs (v0.5.0 is a full stop with a written procedure, and v0.6.0 is one under fleet durability), run under a supervisor, and re-check the compatibility page when you plan a release bump.

This chapter draws on: celld: documentation at v0.6.0 (9 entries) · celld: release notes (5 entries) · Cloudflare documentation (4 entries). The full entries are in the Bibliography.


---

<!-- https://tlockney.github.io/celld-book/building.html -->

## Before you start

celld runs Cloudflare's Workers and Durable Objects programming model on machines you control, with an S3-compatible bucket as the only coordinator. The unit you build with is the **cell**: in Cloudflare terms, a Durable Object. A cell is a small named server with its own private SQLite database and one thread. It serves HTTP, holds WebSockets, sets alarms, and calls out. You make one cell per user, per document, per chat room, per agent. Cells share no database, so the application is sharded from the start. Because the JavaScript API is the Workers API, the same code runs on Cloudflare or on your fleet.

This guide takes you from an empty directory to a running fleet. The companion overview (Part II) explains the architecture, the guarantees, and the tradeoffs in depth; this guide points there when it needs that depth. A few terms recur throughout. A **node** is one celld process. A **fleet** is the set of nodes sharing one **fleet bucket**. Each cell has exactly one **owner** node at a time, recorded in an **ownership record** in the bucket that names the owner and carries a fencing **epoch**. Each node keeps a **node lease** in the bucket, renewed while the node lives. Locally, `celld dev` replaces the bucket with a **local store**.

You need three things on the development machine:

- **A supported platform.**Prebuilt binaries cover Linux x86-64, Linux ARM64, and Apple Silicon. Windows is not supported.
- **esbuild on**celld bundles Worker code with it. Install it with- `PATH`.- `npm i -g esbuild`or- `brew install esbuild`, or point- `CELLD_ESBUILD`at the binary. An asset-only project does not need it.
- **No cloud account yet.**- `celld dev`runs the whole system against the local store. You need a real bucket only when you stand up a fleet in Step 08.

Install the binary, and pin the release you tested:

```
# install (the installer downloads a signed binary to ~/.local/bin)
curl -fsSL https://celld.dev/install.sh | sh
# pin an exact release — rerunning with an older tag is the rollback
# (for the binary only; a v0.5.x or v0.6.x fleet's bucket cannot be served by a pre-v0.5.0 node — see Step 10)
CELLD_VERSION=v0.6.0 sh -c "$(curl -fsSL https://celld.dev/install.sh)"
# optionally verify the GitHub Actions build attestation
gh attestation verify ~/.local/bin/celld --repo denoland/celld
```
## Scaffold the project

A celld application *is* a Wrangler project. There is no celld-specific project format. `celld dev` and `celld deploy` read `wrangler.jsonc` (or `wrangler.json`; **not** `wrangler.toml`) and accept the same layout Cloudflare's tooling does. Start with three files:

```
my-app/
├── wrangler.jsonc
├── src/
│   └── index.ts
├── .dev.vars           # local-only secrets for celld dev — gitignore it
└── .gitignore          # add .celld/ and .dev.vars here
```
The minimal configuration declares the Worker entry point and one Durable Object class:

```
{
  "$schema": "./node_modules/wrangler/config-schema.json",
  "name": "my-app",
  "main": "src/index.ts",
  "compatibility_date": "2026-08-18",
  "durable_objects": {
    "bindings": [{ "name": "COUNTER", "class_name": "Counter" }]
  },
  "migrations": [{ "tag": "v1", "new_sqlite_classes": ["Counter"] }]
}
```
Two details matter here. The `migrations` entry with `new_sqlite_classes` is what gives the class SQLite-backed storage. On celld every cell is SQLite-backed, and this is the declaration that matches Cloudflare's. The binding name `COUNTER` is how the Worker reaches the class through `env`.

The entry point, in the shape of celld's own `counter` example. The default export is the Worker; the exported class is the cell:

```
import { DurableObject } from "cloudflare:workers";
export interface Env {
  COUNTER: DurableObjectNamespace;
}
export class Counter extends DurableObject {
  async fetch(request: Request): Promise<Response> {
    const n = ((await this.ctx.storage.get<number>("n")) ?? 0) + 1;
    await this.ctx.storage.put("n", n);
    return Response.json({ n });
  }
}
export default {
  async fetch(request: Request, env: Env): Promise<Response> {
    const name = new URL(request.url).searchParams.get("name") ?? "default";
    const id = env.COUNTER.idFromName(name);
    return env.COUNTER.get(id).fetch(request);
  },
};
```
`idFromName` is the whole addressing model: the same name maps to the same cell from any node in the fleet, forever. The Worker is the stateless router; the cell is where state lives. TypeScript types come from `@cloudflare/workers-types` (`npm i -D @cloudflare/workers-types`). esbuild strips them at bundle time, so the deploy does not type-check; run `tsc --noEmit` yourself if you want the check.

## Run it locally with celld dev

From the project directory:

```
celld dev                  # Worker listener on http://127.0.0.1:9876
celld dev --port 3000      # pick the port
celld dev --host 0.0.0.0   # expose the Worker listener; internal stays loopback
celld dev --logs           # show the node's info/warning logs too
celld dev --clean          # wipe .celld/dev first — a fresh local store
celld dev --watch-ignore "docs/**"   # extra watcher ignores (repeatable)
celld dev --no-watch       # no automatic builds or restarts (not with --watch-ignore)
```
`celld dev` opens the local store (a SQLite-backed object store), deploys the application, and starts one node. No Docker, no cloud bucket, no configuration. It also reads a `.dev.vars` file beside the Wrangler config, exactly as `wrangler dev` does: one `NAME=value` per line (quotes stripped, no other dotenv features). Those values override same-named `vars`, reload on change, and are never carried to a fleet by `celld deploy`. Then exercise it:

```
$ curl "http://127.0.0.1:9876/?name=alpha"
{"n":1}
$ curl "http://127.0.0.1:9876/?name=alpha"
{"n":2}
$ curl "http://127.0.0.1:9876/?name=beta"
{"n":1}
```
Two names, two cells, two independent SQLite databases. That is the model working.

Lab 1, Part 5 runs this loop live: new code adopted over old state, then a syntax error that fails the rebuild while the last good deployment keeps serving.

The rules of the loop:

- **State lives in**under the project. It survives a normal shutdown;- `.celld/dev`- `celld dev --clean`resets it. A configuration change is- *not*migrated into stored state, so a renamed class or binding against an old store can fail confusingly. Clean and start again. Add- `.celld/`and- `.dev.vars`to- `.gitignore`.
- **The watcher ignores**- `.celld`,- `.wrangler`,- `.git`,- `node_modules`, and- `target`, plus any- `--watch-ignore`globs. It does not follow sources outside the project directory, and it treats a read as no change, so a tool that only reads the project never triggers a build.- `--no-watch`disables automatic builds entirely.
- **The local store is dev-only.**Fleet nodes and operator subcommands require a qualified cloud bucket; the local store is not selectable for them.
- **The internal (operator) listener stays on loopback**even with- `--host 0.0.0.0`, so another machine can reach your app but not the operator API.

## Give the cell real state

The counter used the key-value face of cell storage. Every cell also carries a full SQLite database, synchronous from the cell's point of view: a storage operation never interleaves with another request, so there is no transaction ceremony for ordinary work. A chat room shows all three capabilities at once: SQL state, hibernatable WebSockets, and a durable alarm.

```
import { DurableObject } from "cloudflare:workers";
export class ChatRoom extends DurableObject {
  async fetch(request: Request): Promise<Response> {
    const pair = new WebSocketPair();
    const [client, server] = Object.values(pair);
    this.ctx.acceptWebSocket(server); // hibernatable — the room can sleep
    return new Response(null, { status: 101, webSocket: client });
  }
  async webSocketMessage(ws: WebSocket, message: string | ArrayBuffer) {
    this.ctx.storage.sql.exec(
      "CREATE TABLE IF NOT EXISTS log (at INTEGER, body TEXT)",
    );
    this.ctx.storage.sql.exec(
      "INSERT INTO log (at, body) VALUES (?, ?)",
      Date.now(),
      String(message),
    );
    for (const peer of this.ctx.getWebSockets()) {
      if (peer !== ws) peer.send(message); // one room, no message bus
    }
  }
  async alarm() {
    // prune history older than a day, then reschedule
    this.ctx.storage.sql.exec(
      "DELETE FROM log WHERE at < ?",
      Date.now() - 86_400_000,
    );
    await this.ctx.storage.setAlarm(Date.now() + 3_600_000);
  }
}
```
A cell is in one of three states. It is **resident** while it is in memory on its owner, **hibernated** when celld has evicted it from memory while its hibernatable WebSocket clients stay connected and it stays on its node, and **inactive** when no node holds it and it is only an object in the bucket. The rules that make this shape work in production follow from those states:

- **Keep the constructor cheap.**A cell keeps no memory across state transitions. The constructor runs again on every wake, including every message to a hibernated room. A schema-version check inside- `blockConcurrencyWhile()`belongs there; restoring state does not, so restore from storage inside the handler.
- **Hibernation is the economics.**- `acceptWebSocket()`(rather than the addEventListener API) lets celld evict an idle room from memory while every client stays connected. A thousand quiet rooms cost almost nothing; a message wakes the one room it addresses.
- **Alarms are durable and covered by the acknowledgement gate.**Lab 1, Part 4 shows an alarm set by a request firing on schedule and clearing its pending time. When a handler sets an alarm, celld does not send the successful response until a durable wake entry covers it, so a node loss after the response cannot lose the alarm. A failed- `alarm()`is retried with backoff.
- **Batch WebSocket messages.**Every frame costs a context switch through the cell's one thread. Pack small logical messages into one frame with an envelope format. The reverse case works too: the- **output gate**(the mechanism that holds a cell's outbound effects until the writes they depend on are durable) holds each outbound frame only for its own proof, so a- `webSocketMessage()`handler that sends a frame and then awaits delivers it while still running. You can stream an answer through one handler.
- **Outbound WebSockets do not survive a move.**An outbound socket keeps the cell resident and dies when the cell changes nodes. Keep connection- *intent*in storage and re-dial after activation.
- **Finish write cursors before you answer.**Outside an explicit transaction, a SQL write cursor (an unconsumed- `RETURNING`, say) must complete before a response, an outbound effect, or- `storage.sync()`; celld rejects the output with an error otherwise. A transaction or- `blockConcurrencyWhile()`has a 30-second limit, and a timeout resets the object. Pending I/O after a handler returns stays alive without- `ctx.waitUntil()`.

## Add KV and Queues

Cells cover per-entity state. The first two cross-cutting services, both implemented as cells underneath, are Workers KV for shared key-value data and Queues for decoupled work. Both are graded *Yes* on the compatibility page. Each has its own page under celld.dev/docs/services with a narrative, a worked example, and a short "differences from Cloudflare" list. The differences below are the complete list.

### KV: a durable store, not a CDN

```
"kv_namespaces": [
  { "binding": "SESSIONS", "id": "sessions-prod" }
]
```
```
// in the Worker or any cell
await env.SESSIONS.put(`session:${token}`, JSON.stringify(claims), {
  expirationTtl: 3600,
});
const raw = await env.SESSIONS.get(`session:${token}`);
```
The API is Cloudflare's, with celld's storage model behind it. What changes in practice:

- **No edge cache.**- `cacheTtl`has no effect and- `cacheStatus`is- `null`. Reads route to the namespace's cell. Treat KV as a durable store with KV's API, not as a CDN. Lab 2, Part 2 shows- `cacheStatus: null`on every read and names the namespace's own cell from the node log.
- **One writer per namespace.**Write capacity scales by adding namespaces, not by writing harder to one. The- `id`accepts Cloudflare's hex form or any stable string;- `sessions-prod`is fine.
- **Values above 1 MiB**go to the fleet bucket (small values live in the cell). They are stored under the namespace's ownership epoch, so a superseded owner can never delete the current value. Cloudflare's limits apply: keys up to 512 bytes, values up to 25 MiB, metadata up to 1,024 bytes, a minimum TTL of 60 seconds, and 1,000 keys per- `list()`.- `put()`accepts a- `ReadableStream`value and checks the size limit while it reads.
- **Operate from the CLI:**- `celld kv get/put/delete/list`plus- `bulk`variants that use the Wrangler file format, so- `wrangler kv bulk get`output feeds- `celld kv bulk put`directly.- `celld kv list`pages at 1,000 keys and prints the- `--after`cursor on stderr; data goes to stdout, so pipes carry only data.- `celld kv bulk get`streams rows instead of holding the namespace in memory. A named output file is swapped in atomically when the export completes, but a failed stdout export leaves a truncated JSON array behind.

### Queues: one writer, one consumer, four days

```
"queues": {
  "producers": [{ "binding": "OUTBOX", "queue": "outbox" }],
  "consumers": [
    { "queue": "outbox", "max_batch_size": 10, "max_batch_timeout": 5,
      "max_retries": 2, "dead_letter_queue": "outbox-dead-letter" },
    { "queue": "outbox-dead-letter" }
  ]
}
```
```
// one script can be both the HTTP ingress and the consumer (the examples/queues shape)
export default {
  async fetch(request: Request, env: Env): Promise<Response> {
    await env.OUTBOX.send({ kind: "email", to: "a@example.com", template: "welcome" });
    return new Response("queued", { status: 202 });
  },
  async queue(batch: MessageBatch, env: Env): Promise<void> {
    for (const msg of batch.messages) {
      try {
        await deliver(msg.body);
        msg.ack();
      } catch {
        msg.retry({ delaySeconds: 1 }); // after max_retries it moves to the dead-letter queue
      }
    }
  },
};
```
The consumer settings are Cloudflare's. `max_batch_size` defaults to 10 (max 100), `max_batch_timeout` to 5 seconds (max 60), `max_retries` to 3, and `max_concurrency` caps at 250. A batch closes when it fills or when the timeout expires after its oldest ready message. A message is at most 128,000 bytes, a `sendBatch()` at most 100 messages and 256,000 bytes, and `delaySeconds` runs to 86,400. A handler that returns without settling acknowledges everything; one that throws returns the batch to the queue. A retried message becomes visible again after the `delaySeconds` passed to `retry()` or `retryAll()`, or the consumer's `retry_delay` by default. celld adds no exponential backoff; if you want one, compute the delay from `msg.attempts`. When `attempts` passes `max_retries` the message moves to the `dead_letter_queue`, which is an ordinary queue with its own consumer. Without a dead-letter queue, celld deletes the message. On the producer side, `contentType` (`"v8"`, `"json"`, `"text"`, or `"bytes"`) sets a message's encoding, with the `queues_json_messages` compatibility flag choosing the default, and a producer entry's `delivery_delay` sets a default delay for every message of that binding.

celld's Queues carry real constraints, all of the fail-loudly kind:

- **One writer per queue.**Scale with more queues. Within one queue, producer calls share transactions and durability rounds: a broker admits up to 256 concurrent producer calls, commits at most 64 per transaction, and overlaps four proofs. One queue sustained 7,357 sends per second in a 300,000-send soak. Past the admission limit the owner refuses the call with an error the producer can retry. A caught producer error contains- `cell overload: admission refused`, so a Worker can pass the same 503 back to its client.
- **One consumer script per queue.**A deployment in which two scripts consume one queue fails. The consumer script may also export- `fetch()`: the official- `examples/queues`project exports- `fetch`and- `queue`from one script, as above. Lab 2, Part 5 shows a consumer without a- `queue()`handler taking the local node down.
- **Retention is four days, not configurable.**Pull consumers, the Queues HTTP API, dashboard controls, R2 event notifications, and Queue event subscriptions are not available.
- **Overload is explicit.**A saturated cell or queue answers 503 with- `Retry-After: 1`and- `X-Celld-Overload: cell`. Count those as rejected work rather than retrying immediately at a fixed rate.
- **Operate with**- `celld queue info/peek/purge/pause/resume/redrive`.

## Orchestrate with Workflows

Cells model *entities*: named things whose state persists indefinitely. Workflows are the *process* primitive built on top of them: a sequence of steps that ends, with each step's result stored durably so the sequence survives crashes and restarts. Declare the class in the configuration:

```
"workflows": [
  { "binding": "REPORTS", "name": "report-builder", "class_name": "ReportBuilder" }
]
```
A workflow extends `WorkflowEntrypoint` and does its work in `run()`. This is celld's own example, extended one step:

```
import { WorkflowEntrypoint } from "cloudflare:workers";
import type { WorkflowEvent, WorkflowStep } from "cloudflare:workers";
export class ReportBuilder extends WorkflowEntrypoint {
  async run(event: WorkflowEvent<{ url: string }>, step: WorkflowStep) {
    const fetched = await step.do("fetch source", async () => {
      const response = await fetch(event.payload.url);
      if (!response.ok) throw new Error(`source answered ${response.status}`);
      const text = await response.text();
      return { bytes: text.length, lines: text.split("\n").length };
    });
    await step.sleep("cool off", "30 seconds");
    return await step.do("store summary", async () => {
      // step.do callbacks are the only safe home for side effects
      return { ...fetched, storedAt: Date.now() };
    });
  }
}
export default {
  async fetch(request: Request, env: Env): Promise<Response> {
    const url = new URL(request.url);
    if (url.pathname === "/create") {
      const instance = await env.REPORTS.create({
        params: { url: url.searchParams.get("url") ?? "https://example.com" },
      });
      return Response.json({ id: instance.id });
    }
    const id = url.searchParams.get("id");
    if (url.pathname === "/status" && id) {
      const instance = await env.REPORTS.get(id);
      return Response.json(await instance.status());
    }
    return new Response("Use /create?url=URL or /status?id=ID.", { status: 404 });
  },
};
```
The one rule that decides whether your workflow is correct is the replay rule. A running workflow is stored as its steps. After a crash, `run()` is replayed *from the start*: completed steps return their stored results instantly, and everything *outside* a step runs again. This is also how Cloudflare Workflows behave, which is why the compatibility page does not list it as a difference. The discipline is the same on either platform. Lab 3, Part 3 counts the re-execution: across one durable sleep, the top of `run()` ran twice while each step ran once.

The celld-specific contract, from the workflows service page:

- **Retention is yours to choose.**A successful or failed instance is kept for 30 days by default. Each duration in the- `retention`option can be at most 30 days, and- `delete()`removes a completed run.
- **Limits.**A step result, an event payload, and the workflow parameters are each capped at 1 MiB. Work- *outside*a step cannot stay pending longer than 60 seconds. Pass references (an R2 key, a D1 row) between steps, not payloads.
- **Defaults worth knowing.**- `step.do()`retries 5 times with a 10-second delay and exponential backoff, each attempt capped at 10 minutes, and- `NonRetryableError`stops the loop.- `waitForEvent()`times out after 24 hours. An instance within an hour of its next alarm stays resident (- `CELLD_ALARM_RESIDENT_MS`). A- `workflows`entry cannot carry- `schedules`,- `limits`, or a- `script_name`naming another script. The Workflows REST API and- `wrangler workflows`commands do not operate against celld; drive instances through the binding.
- `locationHint`is accepted
- **Lifecycle.**The page lists no difference for- `pause()`,- `resume()`, or- `restart()`, which means celld intends to match Cloudflare's documented behavior for them. It also lists no caveat for- `create()`called with a terminal instance's ID. If you rely on create-once semantics, verify against your installed release rather than assuming either way.
- **Not available:**rollback, sensitive step results, and- `ReadableStream`step results.

## Add D1 and R2

### D1: a database that is a cell

A D1 database on celld is one cell holding one SQLite database, which means it inherits everything cells have: fencing, replication, durable acknowledgement. Declare it, write migrations, apply them:

```
"d1_databases": [
  { "binding": "DB", "database_name": "ledger" }
]
```
```
# migrations are NNNN_description.sql files (.SQL works too),
# applied in numeric order, one transaction per file — exactly Wrangler's rule;
# a custom migrations_dir must be a relative path inside the project
mkdir -p migrations
cat > migrations/0001_init.sql <<'SQL'
CREATE TABLE entries (
  id INTEGER PRIMARY KEY,
  account TEXT NOT NULL,
  amount INTEGER NOT NULL,
  at INTEGER NOT NULL
);
SQL
# against a fleet (the only documented path — celld d1 finds a node through
# the bucket, so it cannot target celld dev's local store, and a dev deploy
# does not apply migrations; run the SQL from the Worker locally, as celld's
# own examples/d1 does with CREATE TABLE IF NOT EXISTS):
celld d1 migrations apply ledger --bucket "$CELLD_BUCKET"
```
```
const { results } = await env.DB
  .prepare("SELECT account, SUM(amount) AS balance FROM entries WHERE account = ? GROUP BY account")
  .bind(account)
  .all();
```
- **One database, one writer.**More capacity comes from more databases, never a bigger one. Per-entity data belongs in the entity's cell; D1 is for the genuinely shared tables.
- **Result caps:**100,000 rows or 32 MiB per binding result.
- **Importing from Cloudflare:**- `wrangler d1 export`, then- `celld d1 execute ledger --file export.sql`. A migration already recorded in the imported history does not run twice.
- **Not available:**- `dump()`, Time Travel, the D1 REST API.- `wrangler d1`commands do not operate against celld.
- **Migrations on**Lab 2, Part 3 confirms that a dev deploy with a migration file leaves no tables until the Worker runs the SQL itself.- `celld dev`.

### R2: your fleet bucket, wearing the R2 API

```
"r2_buckets": [
  { "binding": "FILES", "bucket_name": "files" }
]
```
```
await env.FILES.put(`reports/${id}.json`, JSON.stringify(report));
const obj = await env.FILES.get(`reports/${id}.json`);
if (obj) { const report = await obj.json(); }
```
An R2 binding stores its objects in the fleet bucket under `r2/<bucket_name>/`, which is the fleet bucket earning its name. Use R2 for step results and artifacts that outgrow the 1 MiB Workflow caps. The differences from Cloudflare:

- **No public URL.**celld serves a bucket through the binding only: no public bucket URL, no presigned URL, no S3 endpoint. To publish a file, put a Worker in front of it.
- **Other tools' objects read fine.**An object written by another tool reads through the binding, with its user metadata as- `customMetadata`and its headers as- `httpMetadata`. Write with- `celld r2 put`to get the complete record.- `delete(keys)`takes up to 1,000 keys per call.
- **Not available:**- `ssecKey`and- `jurisdiction`.
- **Size limits.**A conditional write cannot stream a body larger than 8 MiB. Multipart uploads accept no checksum, cannot resume on another node or after a restart, and out-of-order parts buffer at most 256 MiB.
- **Versions.**An object's- `version`is its content ETag, so identical content yields the same version on a store that reports no version of its own.- `celld dev`'s local store numbers each write instead, so identical bytes get different versions there (Lab 2, Part 4). Never use a version to count writes, and check your store before relying on version equality to de-duplicate;- `checksums.md5`is the content hash everywhere.
- **Keys.**Empty key segments count, as on Cloudflare:- `a/b`,- `/a/b`,- `a//b`, and- `a/b/`are four objects. A key with non-ASCII or special characters is stored percent-encoded, so- `list()`orders it (and applies- `startAfter`) by that encoded form. Page with the returned- `cursor`, not- `startAfter`, if the order matters.

The R2 CLI reads the fleet bucket directly and needs no running node. `celld r2 get|head|put|delete|list` replaces `wrangler r2 object`, takes the `bucket_name` (not the binding name), and preserves the binding's object metadata:

```
celld r2 put files reports/2026-09.json --path report.json \
  --content-type application/json --metadata '{"release":"1.2.3"}' \
  --bucket "$CELLD_BUCKET"
celld r2 list files --prefix reports/ --bucket "$CELLD_BUCKET"   # at most 1000 keys
celld r2 get  files reports/2026-09.json --bucket "$CELLD_BUCKET" > report.json
```
## Stand up a fleet

Everything so far ran on the local store. A fleet needs one thing the laptop cannot fake: a bucket that honors conditional writes with read-after-write consistency, because ownership of every cell rests on it. The bucket must also serve exact ranged reads, because a large cell is restored page by page. Qualified: Amazon S3, Cloudflare R2, Google Cloud Storage, Tigris, and Azure Blob Storage. Not qualified: Backblaze B2, Hetzner, DigitalOcean Spaces. MinIO passes the storage test but is not qualified for production. Using R2 as the example:

```
# an R2 bucket + an S3 API token scoped to it
export AWS_ACCESS_KEY_ID=...
export AWS_SECRET_ACCESS_KEY=...
export AWS_REGION=auto
export S3_ENDPOINT=https://ACCOUNT_ID.r2.cloudflarestorage.com
export CELLD_BUCKET=s3://my-fleet        # a /prefix lets fleets share a bucket
```
Google Cloud Storage (`gs://`, Application Default Credentials) and Azure Blob (`az://`, where the bucket name is the container, with exactly one credential family) follow the same shape: set `CELLD_BUCKET` and the platform's own credentials.

Each node probes the store at startup: conditional writes that must succeed and fail in the right places, plus a ranged read that must return exactly the requested bytes. A clear contract violation stops the node at once rather than risk two owners for one cell. An ambiguous failure gets three attempts and then a warning. The probe cannot be switched off; `CELLD_STORAGE_PROBE` is rejected at startup.

A fleet node binds two listeners: the public Worker listener for ingress, and an internal listener for the peer protocol and operator API. The internal plane carries plain HTTP with no content signature. The network is the security boundary, so put it on a private network or an encrypted overlay (WireGuard, Tailscale), and never expose it:

```
celld \
  --bucket "$CELLD_BUCKET" \
  --listen 0.0.0.0:8080 \
  --internal-listen 10.0.0.12:8081 \
  --advertise node-a.internal:8081
```
To grow the fleet, start another node against the same bucket with a distinct internal address. There is no join command and no membership list; nodes find each other through the node leases in the bucket. Run under a supervisor with no restart limit. A systemd unit carries all the operational rules at once:

```
[Unit]
Description=celld node
After=network-online.target
[Service]
EnvironmentFile=/etc/celld/env
ExecStart=/usr/local/bin/celld --listen 0.0.0.0:8080 \
  --internal-listen 10.0.0.12:8081 --advertise node-a.internal:8081
Restart=always
RestartSec=10          # at least one lease lifetime (CELLD_TTL_MS, 10 s)
TimeoutStopSec=120     # must exceed the stop bound (CELLD_SHUTDOWN_TOTAL_MS, 40 s)
                       # plus the drain-token wait derived from it (3/4 → 30 s)
[Install]
WantedBy=multi-user.target
```
Three of those lines are load-bearing:

- `Restart=always`with no attempt limit.
- `RestartSec=10`
- `TimeoutStopSec=120`- `CELLD_SHUTDOWN_TOTAL_MS`alone (3/4 and 5/8 of it). The separate- `CELLD_SHUTDOWN_DRAIN_MS`and- `CELLD_DRAIN_TOKEN_WAIT_MS`variables are rejected at startup.

Kubernetes users: the same three rules map to a restart policy, a backoff, and `terminationGracePeriodSeconds`, with `/.well-known/celld/health` as the readiness probe. Set a rollout deadline too, because a fresh node whose readiness gate never clears stays unhealthy instead of reporting healthy at the gate's timeout.

Run at least two nodes if write latency matters. With one node every durable write waits for a bucket round trip. With two or more, the owner and its **followers** form an **ensemble**: celld answers on a follower's fsync and uploads afterwards, for the same guarantee at roughly 10× lower write latency.

**The fleet balances itself.** A node that joins takes hibernated cells from the nodes holding the most, and the fleet evens out again after a node leaves. One node samples every lease every five seconds (`CELLD_REBALANCE_INTERVAL_MS`; `0` disables) and writes a shared capacity sample. The most-loaded node per unit of weight (`CELLD_PLACEMENT_WEIGHT`, default the CPU count) hands at most 32 hibernated cells per sample to the peer furthest below its share. Only hibernated cells move, so a fleet without idle eviction balances only the cells that hibernate on their own; set `CELLD_IDLE_EVICT_S` if you want resident-but-idle cells to become eligible. A moved cell's parked WebSockets close with code 1012 so clients reconnect to the new owner. `POST /rebalance/pause` and `/resume` on any node's internal listener stop and restart every move fleet-wide.

## Deploy, operate, observe

Deploying to the fleet is one command from the project directory:

`celld deploy . --bucket "$CELLD_BUCKET"`It bundles (esbuild), signs with the fleet secret, records a full SHA-256 digest of every JavaScript and WebAssembly module in the manifest, and uploads. A node verifies each module against its digest before it builds the deployment. Nodes poll `deploy/current.json` every 30 seconds and adopt in place, with no restart, exactly as `celld dev` rehearsed: the new deployment is built beside the old, new requests switch in one step, in-flight requests finish where they started, and resident Durable Objects move at safe points (forced after 60 seconds with WebSocket code 1012, matching Cloudflare). `POST /reload` on the internal listener adopts immediately. One consequence deserves respect: during the adoption window a request on one deployment can call a Durable Object on the other, so **adjacent versions must accept each other's calls**.

The operator's toolbox, all reading the bucket, none taking ownership:

```
# fleet health: leases, reachability, advertised addresses, versions,
# each node's load sample (owned cells, resident cells, RSS, CPU …),
# and the conditional-write + ranged-read storage test
celld diagnose --bucket "$CELLD_BUCKET"
# list Durable Object instances (Class:ID); reserved __ cells are
# D1 databases, KV namespaces, and Workflows. A class argument scopes
# the storage prefix, so --limit then applies to that class alone
celld cell list --bucket "$CELLD_BUCKET"
celld cell list Counter --limit 50 --bucket "$CELLD_BUCKET"
celld cell list --all --json --bucket "$CELLD_BUCKET" |
  jq -r 'select(.reserved | not) | .scope'
# data services
celld d1 migrations apply ledger --bucket "$CELLD_BUCKET"
celld kv list sessions --all --json --bucket "$CELLD_BUCKET" > keys.ndjson
celld queue info outbox --bucket "$CELLD_BUCKET"
```
During a binary rolling update (where the release allows one; see Step 10), stop one node, wait for its replacement to report healthy *and* for every node to show `restoring=0` in `celld diagnose`, then move on. That lets one restart's cold work finish before the next removes more warm capacity. The first healthy response of a fresh node already waits for the fleet to settle (the readiness gate), so the orchestrator needs no separate fleet-level pause. It does need a rollout deadline, because a node that never settles stays unhealthy rather than passing at the gate's timeout.

For an autoscaler, read the node leases or `GET /state` on the internal listener. Both publish `owned_cells`, `resident_cells`, `host_websockets`, `rss_bytes`, `cpu_percent_x100`, `memory_headroom`, `restoring`, and `capacity_waiting`, plus per-script isolate counts (`live`, `live_empty`, `retiring`, `freed`, with V8 heap bytes) and allocator counters. A `live_empty` count that persists past 30 seconds signals a stuck maintenance pass. A positive `capacity_waiting` is the add-a-node signal; scale down only while every remaining node reports headroom and a small restore backlog. The health path stays a plain boolean. The `/evict/<cell>` operator route waits for the eviction and reports the outcome as JSON: `{"ok":true}`, or a `refused` / `cancelled` / `failed` kind with a reason such as `cell_active` or `alarm_imminent`.

Observability is one variable away, and the sink choice is folded into it. `CELLD_OTEL=1` writes spans as Parquet under `telemetry/` in the fleet bucket: every request, cell event, outbound fetch, and `console.log`, trace-joined and queryable with DuckDB directly, no collector required. `CELLD_OTEL=https://collector.internal:4318` sends the same data over OTLP/HTTP to that base URL instead. The separate `CELLD_OTEL_SINK` variable is rejected at startup.

```
CREATE VIEW traces AS SELECT * FROM
  read_parquet('s3://my-fleet/telemetry/traces/*/*/*/*/*/*.parquet');
SELECT name, duration_us, trace_id FROM traces
ORDER BY duration_us DESC LIMIT 20;
```
## Production rules

The rules that keep a celld application healthy, gathered in one place. Most have appeared above; the rest come from the guarantees and limitations pages.

- **One writer each:**per cell, per D1 database, per KV namespace, per queue. Scale by adding entities, never by growing one.
- **Make remote operations idempotent, with stable operation IDs.**celld never retries a proxied call after body transmission begins (it keeps no replay copy), and it retries only an attempt that provably never started. An ambiguous attempt is yours to retry: same operation ID, handler tolerant of the repeat. The same rule covers WebSocket reconnects, which never move a transport between owners.
- **Keep constructors trivial; batch WebSocket frames; restore state in handlers.**
- **Leave headroom.**Balancing moves only hibernated cells and counts cells by node weight, not by what each cell costs. A fleet at its resident limit still has nowhere to put a lost node's cells.
- **Respect the upgrade cliffs.**Some version transitions must be full stops; others may roll. The complete list:- v0.1→v0.2: full stop.
- v0.2.1→v0.3: may roll. Never downgrade without `node-log close: sealed epoch`in the shutdown log.
- v0.3→v0.4: full stop.
- v0.4.0→v0.4.1: may roll, but never start a v0.4.0 binary once the fleet has paged a large cell in. That node can never activate it.
- **v0.4.1→v0.5.0: full stop.**The procedure from the guarantees page: stop application traffic and deployment writers; stop every old node and its supervisor; wait for every lease to expire; back up the bucket and node data directories; make sure no old binary can restart against the bucket (the format marker cannot stop one from writing); then start v0.5.0 on every node with the same names, addresses, and data directories. Startup migrates the alarm wake format before serving.
- v0.5.0→v0.5.1: rolls. Stop one node, wait for its replacement to report healthy, continue.
- **v0.5.1→v0.6.0: full stop under the default**A v0.6.0 node needs its followers to answer log-tail requests in a ranged format that v0.5.1 nodes do not speak, so stop every v0.5.1 node before starting v0.6.0. A fleet on- `fleet`durability.- `CELLD_DURABILITY=bucket`has no followers and can roll.
- **Do not roll back by starting an old binary.**The only rollback is restoring the stopped-fleet backup, which loses writes made after it.
- Appendix C records what each release changed.
 
- **Drop the removed knobs before you upgrade.**celld refuses to start with any of- `CELLD_OUTPUT_GATE`(the write gate is always on),- `CELLD_STORAGE_PROBE`,- `CELLD_SHUTDOWN_DRAIN_MS`,- `CELLD_DRAIN_TOKEN_WAIT_MS`,- `CELLD_WORKER_LOADER`,- `CELLD_MAX_LOADED_WORKERS`,- `CELLD_OTEL_SINK`,- `CELLD_AI_BINDING`,- `CELLD_AI_URL`, or- `CELLD_REBALANCE_BATCH_CELLS`in the environment. Grep- `/etc/celld/env`first.
- **The security boundary is yours.**TLS terminates in your ingress proxy. The internal listener stays on a private network or an encrypted overlay, and its operator API is mostly unauthenticated. celld does not authenticate your application's users; that is application code, plus- `CELLD_MAX_REQUEST_BODY_BYTES`and- `CELLD_MAX_CELL_REQUESTS`for limits.
- **One application per fleet, single-tenant.**Hostile multi-tenancy is explicitly out of scope for the beta.
- **Pin the release; keep operator tooling and binary together.**The operator API can change between releases. Re-read the compatibility page before every version bump.
- **Watch the health path.**It is- `/.well-known/celld/health`. A fleet coming from v0.3 must update load balancers and probes to it.

The through-line of the whole guide: divide the application into named entities from the start, put every side effect inside something durable (a cell's storage, a queue message, a workflow step), and let the bucket, the epochs, and the fail-loudly deploys do what they were built to do.

This chapter draws on: celld: documentation at v0.6.0 (7 entries) · celld: release notes (4 entries). The full entries are in the Bibliography.


---

<!-- https://tlockney.github.io/celld-book/lab-1.html -->

This chapter is an executed notebook: every output below came from a run against celld v0.6.0 on 2026·09·26. To run it yourself, download this notebook and the helper celld_nb.ts into the same folder (the setup cell imports it), or get all three labs and the helper as a zip. Open the notebook in JupyterLab with a Deno kernel; the setup cell installs celld and esbuild if they are missing.

**Written against celld v0.6.0.** Appendix B defines the terms it uses. This notebook covers Chapters 2 and 7 and Chapter 9, Steps 02–04: what a cell is, how it is addressed, why one thread per cell makes storage safe, SQL storage, alarms, and the `celld dev` loop.

## How these notebooks work

celld is a *runtime*, not a library: V8 + SQLite serving the Cloudflare Workers and Durable Objects API. A notebook cannot `import` it. Instead this notebook is a **driver** for a real `celld dev` process, and the code you write here is real code:

- You define Durable Object classes and the Worker in ordinary cells, against a small kernel-side shim for `cloudflare:workers`(`DurableObject`,`fakeCtx()`) from`celld_nb.ts`. That means a cell class can be**run right here in the kernel**against fake storage first.
- `app.ship({ classes, worker, wrangler })`serializes those same classes with- `Function.prototype.toString()`into- `src/index.js`, writes- `wrangler.jsonc`, and waits for celld's hot reload.
- `app.start()`spawns- `celld dev`;- `app.fetch()`/- `app.json()`talk to it;- `stopAll()`shuts it down.

Two rules follow from the `toString()` trick. A shipped class must be **self-contained**: `toString()` captures the class body, not closures over other cells, so pass shared functions via `helpers`. And the kernel has already stripped TypeScript when it ships, so **what celld runs is JavaScript**. The type annotations in cells are for you, and nothing type-checks them.

**Prerequisites.** A Deno kernel (this one), and `celld` + `esbuild` on the machine. The setup cell finds them on `PATH` or installs both into `~/.cache/celld-nb/tools` (override with `CELLD_BIN` / `CELLD_ESBUILD`; `npm` is needed for the esbuild install). Linux x86-64/ARM64 and Apple Silicon only.

## Setup

Import the helper, locate the tools, and name the project. Each notebook uses its own port so they can run side by side. The project directory is `~/.cache/celld-nb/apps/<name>/` (override the root with `CELLD_NB_HOME`). It is deliberately not beside the notebook: a notebook directory may be a network share, and celld's SQLite state must not live on one. Open it after the first `ship()` to see exactly what celld is bundling.

```
import { App, DurableObject, fakeCtx, stopAll, sleep, type DurableObjectNamespace } from "./celld_nb.ts";
const app = new App("cells", 9901);
console.log("project dir:", app.dir);
```
`import { App, DurableObject, fakeCtx, stopAll, sleep, type DurableObjectNamespace } fr …`4 linesproject dir: /home/jovyan/.cache/celld-nb/apps/cells

## Part 1—A cell is a Durable Object (Chapter 2, Chapter 9, Step 02)

A **cell** is a Durable Object: a small server with a name and a private SQLite database. You make one per user, per document, per chat room, per agent. It serves HTTP, holds WebSockets, sets alarms, and makes outbound calls.

The Worker (the default export) is the stateless router; the exported class is the cell. `idFromName(name)` is the whole addressing model: the same name maps to the same cell from any node in the fleet, forever.

Below is celld's own counter example, as a cell in this notebook. `this.ctx.storage` is the cell's storage; `get`/`put` are its key-value face. Before shipping anything, run the class **in the kernel** against `fakeCtx()`, which puts a Map behind the key-value face and a real in-memory SQLite behind the SQL face.

```
interface Env {
  COUNTER: DurableObjectNamespace;
}
class Counter extends DurableObject<Env> {
  async fetch(_request: Request): Promise<Response> {
    const n = ((await this.ctx.storage.get<number>("n")) ?? 0) + 1;
    await this.ctx.storage.put("n", n);
    return Response.json({ n });
  }
}
const worker = {
  async fetch(request: Request, env: Env): Promise<Response> {
    const name = new URL(request.url).searchParams.get("name") ?? "default";
    const id = env.COUNTER.idFromName(name);
    return env.COUNTER.get(id).fetch(request);
  },
};
// The class is real: instantiate it here, in the kernel, with fake storage.
const local = new Counter(fakeCtx("alpha"), {} as Env);
console.log(await (await local.fetch(new Request("http://cell/"))).json());
console.log(await (await local.fetch(new Request("http://cell/"))).json());
```
`interface Env {`24 lines```
{ n: 1 }
{ n: 2 }
```
Now ship it. `wrangler` here is the same `wrangler.jsonc` the guide's Step 02 shows. The binding name `COUNTER` is how the Worker reaches the class through `env`. The `migrations` entry with `new_sqlite_classes` is what gives the class SQLite-backed storage; on celld every cell is SQLite-backed, and this is the declaration that matches Cloudflare's.

`ship()` returns the module it wrote. Read it: it is your cell, minus types, with the real `cloudflare:workers` import on top.

```
const wrangler = {
  durable_objects: { bindings: [{ name: "COUNTER", class_name: "Counter" }] },
  migrations: [{ tag: "v1", new_sqlite_classes: ["Counter"] }],
};
const module = await app.ship({ classes: [Counter], worker, wrangler });
await app.start({ clean: true });   // --clean: start from an empty local store
console.log(module);
```
`const wrangler = {`8 lines```
celld not found — installing v0.6.0 into /home/jovyan/.cache/celld-nb/tools (https://celld.dev/install.sh)
esbuild not found — installing into /home/jovyan/.cache/celld-nb/tools/esbuild with npm
celld  /home/jovyan/.cache/celld-nb/tools/bin/celld  (celld 0.6.0)
esbuild /home/jovyan/.cache/celld-nb/tools/esbuild/node_modules/.bin/esbuild
celld dev ready at http://127.0.0.1:9901  (project /home/jovyan/.cache/celld-nb/apps/cells)
import { DurableObject } from "cloudflare:workers";
export class Counter extends DurableObject {
  async fetch(_request) {
    const n = (await this.ctx.storage.get("n") ?? 0) + 1;
    await this.ctx.storage.put("n", n);
    return Response.json({
      n
    });
  }
}
export default {
  async fetch (request, env) {
    const name = new URL(request.url).searchParams.get("name") ?? "default";
    const id = env.COUNTER.idFromName(name);
    return env.COUNTER.get(id).fetch(request);
  }
};
```
Step 03 of the guide exercises the counter with `curl`. Same thing from here:

```
console.log(await app.json("/?name=alpha"));
console.log(await app.json("/?name=alpha"));
console.log(await app.json("/?name=beta"));
```
`console.log(await app.json("/?name=alpha"));`3 lines```
{ n: 1 }
{ n: 2 }
{ n: 1 }
```
Two names, two cells, two independent SQLite databases. That is the model working. Compare with the in-kernel run above: the same class, the same answers.

## Part 2—One thread per cell (Chapter 2)

The property that makes the model safe without distributed locking, as Chapter 2 states it:

Two requests to the same cell never run at the same instant. A second request can interleave only while the first

awaits, and storage operations are synchronous, so a storage operation never interleaves at all.

The subtle part is *which* awaits let another request in. Awaiting a storage operation does not (Durable Objects call this the input gate). Awaiting anything else does: an outbound `fetch`, a timer, a WebSocket send. So a read-modify-write that awaits a non-storage promise in the middle is a lost-update bug, exactly as it would be against a shared database.

`Racy` below does that deliberately, with a 100 ms timer standing in for an outbound call. `mode=gated` wraps the same work in `blockConcurrencyWhile()`, which holds the cell's input gate for the duration (30-second limit; a timeout resets the object). Fire ten concurrent requests at each.

```
class Racy extends DurableObject<Env> {
  async fetch(request: Request): Promise<Response> {
    const mode = new URL(request.url).searchParams.get("mode") ?? "racy";
    const work = async () => {
      const n = (await this.ctx.storage.get<number>("n")) ?? 0;
      await new Promise((r) => setTimeout(r, 100));   // NOT a storage op: another request may run here
      await this.ctx.storage.put("n", n + 1);
      return n + 1;
    };
    const n = mode === "gated" ? await this.ctx.blockConcurrencyWhile(work) : await work();
    return Response.json({ n });
  }
}
// The Worker now routes by binding name too: /?ns=RACY&name=…
const router = {
  async fetch(request: Request, env: Record<string, DurableObjectNamespace>): Promise<Response> {
    const url = new URL(request.url);
    const ns = env[url.searchParams.get("ns") ?? "COUNTER"];
    if (!ns) return new Response(`no binding ${url.searchParams.get("ns")}`, { status: 404 });
    return ns.get(ns.idFromName(url.searchParams.get("name") ?? "default")).fetch(request);
  },
};
wrangler.durable_objects.bindings.push({ name: "RACY", class_name: "Racy" });
wrangler.migrations.push({ tag: "v2", new_sqlite_classes: ["Racy"] });
await app.ship({ classes: [Counter, Racy], worker: router, wrangler });
const fire = (name: string, mode: string) =>
  Promise.all(Array.from({ length: 10 }, () => app.json<{ n: number }>(`/?ns=RACY&name=${name}&mode=${mode}`)));
console.log("racy  →", (await fire("r", "racy")).map((x) => x.n).join(" "));
console.log("gated →", (await fire("g", "gated")).map((x) => x.n).join(" "));
console.log("counter alpha survived the reload:", await app.json("/?name=alpha"));
```
`class Racy extends DurableObject<Env> {`34 lines```
reloaded cells (1162 bytes)
racy  → 1 1 1 1 1 1 1 1 1 1
gated → 1 10 6 9 3 5 2 4 7 8
counter alpha survived the reload: { n: 3 }
```
Ten racy requests, ten answers of `1`: every request read `0`, yielded at the timer, and wrote `1`. Ten gated requests, the numbers 1–10 in whatever order the responses arrived: each one saw the previous write. Note the third line, too. Adding a class and a migration tag was a hot reload, and `alpha`'s count carried on from Part 1.

The rule that follows: do the read-modify-write between storage operations only, and put outbound work either before the read or after the write. `blockConcurrencyWhile` is the escape hatch, not the default, because it stalls every other request to the cell.

## Part 3—Give the cell real state (Chapter 9, Step 04)

Every cell also carries a full SQLite database, synchronous from the cell's point of view: `this.ctx.storage.sql.exec(query, ...binds)` returns a cursor with `toArray()` and `one()`. There is no transaction ceremony for ordinary work, because nothing interleaves with a storage operation.

Two production rules from Step 04 show up in the shape of `Ledger`:

- **Keep the constructor trivial.**A cell keeps no memory across state transitions, and the constructor runs again on every wake. So the schema is ensured inside the handler (- `CREATE TABLE IF NOT EXISTS`is cheap), and nothing is restored in the constructor.
- **Finish write cursors before you answer.**An unconsumed- `RETURNING`cursor at response time is an error on celld.- `Ledger`consumes its- `RETURNING`with- `one()`immediately.

`fakeCtx()`'s SQL face is real SQLite (`node:sqlite`), so run the ledger in the kernel first.

```
class Ledger extends DurableObject<Env> {
  #ensureSchema() {
    this.ctx.storage.sql.exec(
      "CREATE TABLE IF NOT EXISTS entries (id INTEGER PRIMARY KEY, amount INTEGER NOT NULL, memo TEXT, at INTEGER NOT NULL)",
    );
  }
  async fetch(request: Request): Promise<Response> {
    this.#ensureSchema();
    const url = new URL(request.url);
    if (request.method === "POST" && url.pathname === "/entries") {
      const { amount, memo } = await request.json() as { amount: number; memo?: string };
      const row = this.ctx.storage.sql
        .exec<{ id: number }>("INSERT INTO entries (amount, memo, at) VALUES (?, ?, ?) RETURNING id", amount, memo ?? null, Date.now())
        .one();                                   // consume the write cursor before answering
      return Response.json({ id: row.id });
    }
    if (url.pathname === "/balance") {
      const { balance, count } = this.ctx.storage.sql
        .exec<{ balance: number | null; count: number }>("SELECT SUM(amount) AS balance, COUNT(*) AS count FROM entries")
        .one();
      return Response.json({ balance: balance ?? 0, count });
    }
    if (url.pathname === "/entries") {
      return Response.json(this.ctx.storage.sql.exec("SELECT id, amount, memo, at FROM entries ORDER BY id").toArray());
    }
    return new Response("POST /entries {amount, memo} · GET /balance · GET /entries", { status: 404 });
  }
}
const post = (url: string, body: unknown) =>
  new Request(url, { method: "POST", body: JSON.stringify(body), headers: { "content-type": "application/json" } });
const ledger = new Ledger(fakeCtx("acct-1"), {} as Env);
await ledger.fetch(post("http://cell/entries", { amount: 120, memo: "deposit" }));
await ledger.fetch(post("http://cell/entries", { amount: -45, memo: "coffee" }));
console.log(await (await ledger.fetch(new Request("http://cell/balance"))).json());
console.log(await (await ledger.fetch(new Request("http://cell/entries"))).json());
```
`class Ledger extends DurableObject<Env> {`38 lines```
{ balance: 75, count: 2 }
[
  { id: 1, amount: 120, memo: "deposit", at: 1790463320686 },
  { id: 2, amount: -45, memo: "coffee", at: 1790463320686 }
]
```
Ship it and run the same sequence against celld. The query string carries the routing (`ns`, `name`); the path carries the operation.

```
wrangler.durable_objects.bindings.push({ name: "LEDGER", class_name: "Ledger" });
wrangler.migrations.push({ tag: "v3", new_sqlite_classes: ["Ledger"] });
await app.ship({ classes: [Counter, Racy, Ledger], worker: router, wrangler });
const acct = (path: string, init?: RequestInit) => app.json(`${path}?ns=LEDGER&name=acct-1`, init);
console.log(await acct("/entries", { method: "POST", body: JSON.stringify({ amount: 120, memo: "deposit" }) }));
console.log(await acct("/entries", { method: "POST", body: JSON.stringify({ amount: -45, memo: "coffee" }) }));
console.log(await acct("/balance"));
console.log("a different name is a different database:", await app.json("/balance?ns=LEDGER&name=acct-2"));
```
`wrangler.durable_objects.bindings.push({ name: "LEDGER", class_name: "Ledger" });`9 lines```
reloaded cells (2404 bytes)
{ id: 1 }
{ id: 2 }
{ balance: 75, count: 2 }
a different name is a different database: { balance: 0, count: 0 }
```
## Part 4—Alarms (Chapter 9, Step 04, Chapter 6)

An alarm is a durable wake-up: `setAlarm(when)` in a handler, and celld calls `alarm()` at that time, on whichever node owns the cell then, after a restart, after a move. Two celld guarantees from Step 04 of the guide:

- **Alarms are covered by the acknowledgement gate.**When a handler sets an alarm, celld does not send the successful response until a durable wake entry covers it. A node loss after the response cannot lose the alarm.
- **A failed**- `alarm()`is retried with backoff.

`Reminder` schedules itself `in` milliseconds ahead and records each firing in SQL. In the kernel, `fakeCtx()` only *records* alarms (`ctx.alarms`); nothing fires, which is enough to unit-test the scheduling logic. On celld it fires for real.

```
class Reminder extends DurableObject<Env> {
  async fetch(request: Request): Promise<Response> {
    this.ctx.storage.sql.exec("CREATE TABLE IF NOT EXISTS firings (at INTEGER NOT NULL, note TEXT)");
    const url = new URL(request.url);
    if (url.pathname === "/remind") {
      const inMs = Number(url.searchParams.get("in") ?? 1000);
      await this.ctx.storage.put("note", url.searchParams.get("note") ?? "ping");
      await this.ctx.storage.setAlarm(Date.now() + inMs);
      return Response.json({ scheduledFor: await this.ctx.storage.getAlarm() });
    }
    return Response.json({
      pending: await this.ctx.storage.getAlarm(),
      firings: this.ctx.storage.sql.exec("SELECT at, note FROM firings ORDER BY at").toArray(),
    });
  }
  async alarm(): Promise<void> {
    this.ctx.storage.sql.exec("CREATE TABLE IF NOT EXISTS firings (at INTEGER NOT NULL, note TEXT)");
    const note = await this.ctx.storage.get<string>("note");
    this.ctx.storage.sql.exec("INSERT INTO firings (at, note) VALUES (?, ?)", Date.now(), note ?? null);
  }
}
// In the kernel: the alarm is recorded, not fired — call alarm() by hand to test its body.
const ctx = fakeCtx("r1");
const reminder = new Reminder(ctx, {} as Env);
await reminder.fetch(new Request("http://cell/remind?in=500¬e=kettle"));
console.log("recorded alarms:", ctx.alarms.length, "→ pending:", await ctx.storage.getAlarm());
await reminder.alarm();
console.log(await (await reminder.fetch(new Request("http://cell/"))).json());
```
`class Reminder extends DurableObject<Env> {`30 lines```
recorded alarms: 1 → pending: 1790463322991
{
  pending: 1790463322991,
  firings: [ { at: 1790463322491, note: "kettle" } ]
}
```
```
wrangler.durable_objects.bindings.push({ name: "REMINDER", class_name: "Reminder" });
wrangler.migrations.push({ tag: "v4", new_sqlite_classes: ["Reminder"] });
await app.ship({ classes: [Counter, Racy, Ledger, Reminder], worker: router, wrangler });
const rem = (path: string) => app.json<{ pending: number | null; firings: unknown[] }>(`${path}${path.includes("?") ? "&" : "?"}ns=REMINDER&name=r1`);
console.log(await rem("/remind?in=1500¬e=kettle"));
console.log("right away:", await rem("/"));
await sleep(2500);
console.log("2.5 s later:", await rem("/"));
```
`wrangler.durable_objects.bindings.push({ name: "REMINDER", class_name: "Reminder" });`9 lines```
reloaded cells (3426 bytes)
{ scheduledFor: 1790463329134 }
right away: { pending: 1790463329134, firings: [] }
2.5 s later: { pending: null, firings: [ { at: 1790463329136, note: "kettle" } ] }
```
`pending` is the alarm's timestamp while it is scheduled and `null` after it fires; the firing itself is a row that survived the wake. Nothing in memory did. `alarm()` re-creates the table because, on a hibernated or moved cell, it may be the first code that runs after the constructor.

## Part 5—The `celld dev` loop (Chapter 9, Step 03)

You have already used the loop four times without looking at it. Editing a source file triggers the watcher, which builds a new deployment *beside* the running one and adopts it in place. Durable state in `.celld/dev` persists across rebuilds and normal shutdowns. A failed build leaves the current application serving. The dev loop is the production deployment mechanism in miniature.

Two demonstrations. First, redeclare `Counter` with a `step` parameter and re-ship: `alpha` continues from its old count under the new code. Second, write a broken `src/index.js` by hand and watch what celld does.

```
class Counter extends DurableObject<Env> {
  async fetch(request: Request): Promise<Response> {
    const step = Number(new URL(request.url).searchParams.get("step") ?? 1);
    const n = ((await this.ctx.storage.get<number>("n")) ?? 0) + step;
    await this.ctx.storage.put("n", n);
    return Response.json({ n, step });
  }
}
await app.ship({ classes: [Counter, Racy, Ledger, Reminder], worker: router, wrangler });
console.log("alpha, new code, old state:", await app.json("/?name=alpha&step=10"));
// Now break the build on purpose. ship() would refuse; write the file directly.
app.clearLogs();
await Deno.writeTextFile(`${app.dir}/src/index.js`, "export default { fetch( { this is not JavaScript");
await sleep(4000);
console.log(app.logs({ grep: /ERROR|error|change detected/ }));
console.log("still serving the last good build:", await app.json("/?name=alpha&step=0"));
// …and put it back.
await app.ship({ classes: [Counter, Racy, Ledger, Reminder], worker: router, wrangler });
console.log("restored:", await app.json("/?name=alpha&step=0"));
```
`class Counter extends DurableObject<Env> {`21 lines```
reloaded cells (3517 bytes)
alpha, new code, old state: { n: 13, step: 10 }
  ● change detected; rebuilding the application
  error reload failed: esbuild failed:
✘ [ERROR] Expected "}" but found "is"
1 error
still serving the last good build: { n: 13, step: 0 }
reloaded cells (3517 bytes)
restored: { n: 13, step: 0 }
```
The rules of the loop, from Step 03:

- State lives in `.celld/dev`under the project, and`--clean`resets it.
- A configuration change is *not*migrated into stored state, so a renamed class or binding against an old store can fail confusingly. Clean and start again.
- The watcher ignores `.celld`,`.wrangler`,`.git`,`node_modules`, and`target`.

### Exercise—a rate limiter cell

One cell per API key is the canonical Durable Object rate limiter: the cell *is* the partition, so there is no shared counter to contend on. Implement `RateLimiter` as a token bucket:

- `capacity`tokens (default 3), refilled at- `refillPerSec`(default 1). Both read from the query string so the checker can vary them.
- `GET /`takes one token: respond- `200 {"ok":true,"tokens":<remaining>}`or- `429 {"ok":false,"retryInMs":<ms until the next token>}`.
- Keep `tokens`and`lastRefill`in storage (`get`/`put`or SQL, your choice).**No timers**: compute the refill from elapsed time on each request. That is what lets the cell hibernate for free between calls.
- Respect Part 2: no non-storage `await`between reading and writing the bucket.

The checker runs your class in the kernel with `fakeCtx()`, then ships it and runs the same sequence on celld.

**Hints**, from a nudge to a near-solution. Try the exercise first, then read only as far as you need.

- **Concept.**A token bucket needs no timer. On each request, work out how many tokens the time since- `lastRefill`has earned, add them (capped at- `capacity`), and only then decide.
- **Data shape.**One stored value is enough:- `{ tokens, lastRefill }`under a single key with- `this.ctx.storage.get`and- `put`, defaulting to- `{ tokens: capacity, lastRefill: Date.now() }`on first use. Tokens can be fractional; spend one only when- `tokens >= 1`.
- **Sketch.**- `tokens = min(capacity, tokens + refillPerSec × elapsedSeconds)`, then- `lastRefill = now`. With a token to spend, subtract one, store, and answer 200 with the whole tokens left. Without one, store and answer 429 with- `retryInMs`= the time the missing fraction of a token takes at- `refillPerSec`. Every- `await`on that path is a storage call.

Full worked solutions are intentionally not included. The checker cell is the verification: every line turns from ✗ to ✓ once the implementation is right.

```
class RateLimiter extends DurableObject<Env> {
  async fetch(request: Request): Promise<Response> {
    const url = new URL(request.url);
    const capacity = Number(url.searchParams.get("capacity") ?? 3);
    const refillPerSec = Number(url.searchParams.get("refillPerSec") ?? 1);
    // TODO: read {tokens, lastRefill} from storage (default: full bucket, now),
    //       add refillPerSec * elapsedSeconds (capped at capacity), then
    //       either spend one token and answer 200, or answer 429 with retryInMs.
    return new Response("not implemented", { status: 501 });
  }
}
```
`class RateLimiter extends DurableObject<Env> {`11 lines```
// ── checker ────────────────────────────────────────────────────────────────
const burst = async (call: (q: string) => Promise<Response>, n: number) => {
  const out: string[] = [];
  for (let i = 0; i < n; i++) out.push(String((await call("capacity=3&refillPerSec=2")).status));
  return out.join(" ");
};
const rl = new RateLimiter(fakeCtx("key-1"), {} as Env);
const inKernel = await burst((q) => rl.fetch(new Request(`http://cell/?${q}`)), 5);
console.log("kernel:", inKernel, inKernel === "200 200 200 429 429" ? "✓" : "✗ expected 200 200 200 429 429");
await sleep(1100);
const after = (await rl.fetch(new Request("http://cell/?capacity=3&refillPerSec=2"))).status;
console.log("after 1.1 s at 2 tokens/s:", after, after === 200 ? "✓" : "✗ expected 200 (refilled)");
wrangler.durable_objects.bindings.push({ name: "LIMITER", class_name: "RateLimiter" });
wrangler.migrations.push({ tag: "v5", new_sqlite_classes: ["RateLimiter"] });
await app.ship({ classes: [Counter, Racy, Ledger, Reminder, RateLimiter], worker: router, wrangler });
const onCelld = await burst((q) => app.fetch(`/?ns=LIMITER&name=key-1&${q}`), 5);
console.log("celld: ", onCelld, onCelld === "200 200 200 429 429" ? "✓" : "✗");
console.log("another key is another bucket:", (await app.fetch("/?ns=LIMITER&name=key-2&capacity=3&refillPerSec=2")).status);
```
`const burst = async (call: (q: string) => Promise<Response>, n: number) => {`20 lineskernel: 501 501 501 501 501 ✗ expected 200 200 200 429 429 after 1.1 s at 2 tokens/s: 501 ✗ expected 200 (refilled) reloaded cells (4088 bytes) celld: 501 501 501 501 501 ✗ another key is another bucket: 501

## Clean up

Stop the node. State stays in the project's `.celld/dev`; the next `app.start()` without `clean: true` resumes it.

`await stopAll();``await stopAll();`1 linecelld dev stopped (exit 0)

## Where next

That is the cell model from Chapters 2 and 7 and Steps 02–04 of the guide: a named object with one thread and its own SQLite, addressed by `idFromName`, safe to mutate because storage never interleaves, with durable alarms and a dev loop that keeps state across deploys.

- **Lab 2—Bindings**adds the cross-cutting services (Steps 05 and 07): KV, Queues, D1, and R2, all cells underneath.
- **Lab 3—Processes and lifecycle**covers Workflows (Step 06), WebSockets and hibernation, cron triggers, RPC, and reading celld's lifecycle events.

What these notebooks deliberately leave to the articles: fleets, buckets, ownership and fencing (Chapter 3–05, Steps 08–10). Those are operational, and `celld dev`'s local store is not a fleet.

**Maintenance.** This export is frozen at celld v0.6.0, executed 2026·09·26. Re-execute it on each new celld release and note what changed: upstream moves fast, with five releases (v0.4.0 through v0.6.0) between 2026·08·28 and 2026·09·26.

This lab draws on: celld: documentation at v0.6.0 (3 entries). The full entries are in the Bibliography.


---

<!-- https://tlockney.github.io/celld-book/lab-2.html -->

This chapter is an executed notebook: every output below came from a run against celld v0.6.0 on 2026·09·26. To run it yourself, download this notebook and the helper celld_nb.ts into the same folder (the setup cell imports it), or get all three labs and the helper as a zip. Open the notebook in JupyterLab with a Deno kernel; the setup cell installs celld and esbuild if they are missing.

**Written against celld v0.6.0.** Appendix B defines the terms it uses. This notebook covers Chapter 9, Steps 05 and 07 and the KV, Queues, R2, Wrangler-configuration, and "D1 is a cell" subsections of Chapter 6: the four cross-cutting services, what each one is underneath, and the two places where `celld dev` behaves differently from a fleet. Each service also has its own page under celld.dev/docs/services, with a worked example.

**How this notebook works** (Lab 1 has the full explanation): cells define the Worker and any Durable Object classes as real TypeScript; `app.ship({ classes, worker, wrangler, files })` serializes them with `Function.prototype.toString()` into `src/index.js` + `wrangler.jsonc` and waits for celld's hot reload; `app.start()` spawns `celld dev`; `app.json()` / `app.text()` / `app.fetch()` talk to it; `app.logs()` reads the node's output. Shipped code must be self-contained (no closures over other cells), and what celld runs is JavaScript—the types are for you.

Unlike the cells of Lab 1, **none of these four bindings has an in-kernel fake**: every cell below runs against `celld dev`.

## Setup

Import the helper and name the project. Port 9902 keeps this notebook clear of the other two; the project directory is printed below (`app.dir`, under the root Lab 1 describes). The helper types Durable Object bindings only, so the second block declares the minimal shapes of the four services this notebook uses. They are enough to type `env` in cells; nothing checks them, and they are not shipped.

```
import { App, DurableObject, stopAll, sleep, type DurableObjectNamespace } from "./celld_nb.ts";
import { DatabaseSync } from "node:sqlite";
interface KVNamespace {
  get(key: string, type?: "text" | "json" | { cacheTtl?: number }): Promise<unknown>;
  getWithMetadata(key: string, opts?: { cacheTtl?: number }): Promise<unknown>;
  put(key: string, value: string, opts?: { expirationTtl?: number; expiration?: number; metadata?: unknown }): Promise<void>;
  list(opts?: { prefix?: string; limit?: number }): Promise<unknown>;
  delete(key: string): Promise<void>;
}
interface D1Result { success: boolean; results: Record<string, unknown>[]; meta: Record<string, unknown> }
interface D1Statement {
  bind(...values: unknown[]): D1Statement;
  all(): Promise<D1Result>;
  first(column?: string): Promise<unknown>;
  run(): Promise<D1Result>;
  raw(): Promise<unknown[][]>;
}
interface D1Database {
  prepare(sql: string): D1Statement;
  batch(statements: D1Statement[]): Promise<D1Result[]>;
  exec(sql: string): Promise<{ count: number; duration: number }>;
  dump(): Promise<ArrayBuffer>;
}
interface R2Object {
  key: string; size: number; version: string; etag: string; httpEtag: string; uploaded: Date;
  httpMetadata?: { contentType?: string }; customMetadata?: Record<string, string>;
  checksums: { toJSON(): Record<string, string> };
  text(): Promise<string>; json(): Promise<unknown>;
}
interface R2Bucket {
  put(key: string, body: string, opts?: Record<string, unknown>): Promise<R2Object | null>;
  get(key: string): Promise<R2Object | null>;
  head(key: string): Promise<R2Object | null>;
  list(opts?: { prefix?: string; include?: string[] }): Promise<{ objects: R2Object[]; truncated: boolean }>;
  delete(key: string): Promise<void>;
}
interface QueueMessage { id: string; attempts: number; body: Record<string, unknown>; ack(): void; retry(opts?: { delaySeconds?: number }): void }
interface MessageBatch { queue: string; messages: QueueMessage[]; ackAll(): void; retryAll(): void }
interface Queue { send(body: unknown): Promise<void>; sendBatch(messages: { body: unknown }[]): Promise<void> }
interface Env {
  SESSIONS: KVNamespace;
  DB: D1Database;
  FILES: R2Bucket;
  OUTBOX: Queue;
  PROFILE: DurableObjectNamespace;
}
const app = new App("bindings", 9902);
console.log("project dir:", app.dir);
```
`import { App, DurableObject, stopAll, sleep, type DurableObjectNamespace } from "./cel …`51 linesproject dir: /home/jovyan/.cache/celld-nb/apps/bindings

## Part 1—The theme: every service is a cell (Chapter 9, Step 05, Chapter 6 "Wrangler configuration")

Cells cover per-entity state. The four services in this notebook are the *cross-cutting* ones, data shared across entities, and the articles describe each with the same sentence: it is a cell underneath. A KV namespace is a cell. A queue is a cell. A D1 database is "one cell holding one SQLite database". An R2 bucket is a prefix in the fleet bucket. So each inherits what cells have (fencing, replication, durable acknowledgement), and each inherits the cell's limit: **one writer per namespace / queue / database.** Capacity comes from adding more of them, never from growing one.

Bindings are declared in `wrangler.jsonc` and reach the Worker (and any cell) through `env`. celld models a fixed set of top-level keys and, by its fail-loudly rule, stops a deploy on any key it does not model. The accepted keys (Step 02):


`$schema`,`name`,`main`,`no_bundle`,`compatibility_date`,`compatibility_flags`,`durable_objects`,`migrations`,`assets`,`services`,`triggers`,`vars`,`d1_databases`,`kv_namespaces`,`queues`,`workflows`,`r2_buckets`,`worker_loaders`,`containers`,`define`, and`rules`.`define`and`rules`are passed to the esbuild run, so they cannot combine with`no_bundle`.

Below: the four declarations from the articles in one configuration, a Worker that only reports what `env` holds, and the binding table celld prints at deploy. The queue's `consumers` entry is left out until Part 5, for a reason that Part shows.

```
const wrangler: Record<string, unknown> = {
  kv_namespaces: [{ binding: "SESSIONS", id: "sessions-prod" }],
  d1_databases: [{ binding: "DB", database_name: "ledger" }],
  r2_buckets: [{ binding: "FILES", bucket_name: "files" }],
  queues: { producers: [{ binding: "OUTBOX", queue: "outbox" }] },
};
const envWorker = {
  async fetch(_request: Request, env: Record<string, unknown>): Promise<Response> {
    const shape: Record<string, string> = {};
    for (const [name, binding] of Object.entries(env)) shape[name] = binding?.constructor?.name ?? typeof binding;
    return Response.json(shape);
  },
};
await app.ship({ worker: envWorker, wrangler });
await app.start({ clean: true, logs: true });   // --logs: the node's INFO lines show each service's cell being born
console.log(app.logs({ grep: /^(Binding|env\.)/ }));
console.log(await app.json("/"));
```
`const wrangler: Record<string, unknown> = {`19 lines```
celld  /home/jovyan/.cache/celld-nb/tools/bin/celld  (celld 0.6.0)
esbuild /home/jovyan/.cache/celld-nb/tools/esbuild/node_modules/.bin/esbuild
celld dev ready at http://127.0.0.1:9902  (project /home/jovyan/.cache/celld-nb/apps/bindings)
Binding                 Resource
env.DB (D1)             ledger
env.SESSIONS (KV)       sessions-prod
env.OUTBOX (Queue)      outbox
env.FILES (R2)          files
{
  FILES: "Object",
  SESSIONS: "KvNamespace",
  OUTBOX: "Queue",
  DB: "D1Database"
}
```
Four bindings, four resources, and `env` holds one object per binding. Now the fail-loudly rule, on the key the article names. `ship()` would happily write any key, so put `routes` into `wrangler.jsonc` by hand and watch the reload.

```
app.clearLogs();
const current = JSON.parse(await Deno.readTextFile(`${app.dir}/wrangler.jsonc`));
await Deno.writeTextFile(`${app.dir}/wrangler.jsonc`, JSON.stringify({ ...current, routes: ["example.com/*"] }, null, 2));
await sleep(3000);
console.log(app.logs({ grep: /reload/ }));
console.log("still serving the last good deployment:", (await app.fetch("/")).status);
await app.ship({ worker: envWorker, wrangler });   // put the good configuration back
console.log("restored:", (await app.fetch("/")).status);
```
`app.clearLogs();`8 lineserror reload failed: `celld deploy` does not support these config keys: routes. still serving the last good deployment: 200 reloaded bindings (225 bytes) restored: 200

The error names the key, the previous deployment keeps serving, and a good `ship()` afterwards reloads normally. Keep the message in mind: the *same* rule applies to binding entries celld does not model, and Part 5 meets a case where it costs more than a failed reload.

## Part 2—KV: a durable store, not a CDN (Chapter 9, Step 05, Chapter 6 "KV")

`"kv_namespaces": [{ "binding": "SESSIONS", "id": "sessions-prod" }]`The API is Cloudflare's (`put` / `get` / `getWithMetadata` / `list` / `delete`, `expirationTtl` and `expiration`, `metadata`), with celld's storage model behind it. The article's list of what changes in practice:

- **No edge cache.**- `cacheTtl`has no effect and- `cacheStatus`is- `null`. "Reads route to the namespace's cell. Treat KV as a durable store with KV's API, not as a CDN."
- **One writer per namespace.**The- `id`accepts Cloudflare's hex form or any stable string;- `sessions-prod`is fine.
- Values above 1 MiB go to the fleet bucket; small values live in the cell.

Two readers below: the Worker, through the routes, and a `Profile` cell that reads the same namespace through `this.env` while keeping its own per-entity counter in its own storage. That is the division of labour the article prescribes.

```
class Profile extends DurableObject<Env> {
  async fetch(request: Request): Promise<Response> {
    const token = new URL(request.url).searchParams.get("token") ?? "";
    const session = await this.env.SESSIONS.get(`session:${token}`, "json");   // shared data: the namespace
    const visits = ((await this.ctx.storage.get<number>("visits")) ?? 0) + 1;  // per-entity data: this cell
    await this.ctx.storage.put("visits", visits);
    return Response.json({ cell: this.ctx.id.name, session, visits });
  }
}
const kvWorker = {
  async fetch(request: Request, env: Env): Promise<Response> {
    const url = new URL(request.url);
    const q = (name: string) => url.searchParams.get(name);
    const key = q("key") ?? "";
    try {
      switch (url.pathname) {
        case "/put": {
          const opts: { expirationTtl?: number; metadata?: unknown } = {};
          if (q("ttl")) opts.expirationTtl = Number(q("ttl"));
          if (q("meta")) opts.metadata = JSON.parse(q("meta") ?? "null");
          await env.SESSIONS.put(key, q("value") ?? "", opts);
          return Response.json({ put: key, ...opts });
        }
        case "/get":
          return Response.json(await env.SESSIONS.getWithMetadata(key, { cacheTtl: 3600 }));   // cacheTtl: no effect
        case "/list":
          return Response.json(await env.SESSIONS.list({ prefix: q("prefix") ?? "" }));
        case "/delete":
          await env.SESSIONS.delete(key);
          return Response.json({ deleted: key });
        case "/profile":
          return env.PROFILE.get(env.PROFILE.idFromName(q("name") ?? "anon")).fetch(request);
      }
      return new Response("not found", { status: 404 });
    } catch (e) {
      return Response.json({ error: String(e) }, { status: 500 });
    }
  },
};
wrangler.durable_objects = { bindings: [{ name: "PROFILE", class_name: "Profile" }] };
wrangler.migrations = [{ tag: "v1", new_sqlite_classes: ["Profile"] }];
await app.ship({ classes: [Profile], worker: kvWorker, wrangler });
const claims = encodeURIComponent(JSON.stringify({ user: "ada", role: "admin" }));
console.log(await app.json(`/put?key=session:abc&value=${claims}&ttl=3600&meta=${encodeURIComponent('{"ua":"notebook"}')}`));
console.log(await app.json("/put?key=session:def&value=plain&ttl=3600"));
console.log(await app.json("/put?key=flag:beta&value=on"));
console.log("get     →", await app.json("/get?key=session:abc"));
console.log("list    →", await app.json("/list?prefix=session:"));
console.log("cell    →", await app.json("/profile?name=ada&token=abc"));
console.log("cell    →", await app.json("/profile?name=ada&token=nope"));
console.log("delete  →", await app.json("/delete?key=flag:beta"), await app.json("/get?key=flag:beta"));
console.log("ttl<60  →", await app.json("/put?key=short&value=x&ttl=5"));
console.log(app.logs({ grep: /cell_isolate_startup_timing.*__KvNamespace/, last: 1 }).replace(/^.*scope=/, "the namespace's cell: scope=").replace(/ node=.*$/, ""));
```
`class Profile extends DurableObject<Env> {`56 lines```
reloaded bindings (1886 bytes)
{
  put: "session:abc",
  expirationTtl: 3600,
  metadata: { ua: "notebook" }
}
{ put: "session:def", expirationTtl: 3600 }
{ put: "flag:beta" }
get     → {
  value: '{"user":"ada","role":"admin"}',
  metadata: { ua: "notebook" },
  cacheStatus: null
}
list    → {
  keys: [
    {
      name: "session:abc",
      metadata: { ua: "notebook" },
      expiration: 1790466960
    },
    { name: "session:def", expiration: 1790466960 }
  ],
  list_complete: true,
  cacheStatus: null
}
cell    → { cell: "ada", session: { user: "ada", role: "admin" }, visits: 1 }
cell    → { cell: "ada", session: null, visits: 2 }
delete  → { deleted: "flag:beta" } { value: null, metadata: null, cacheStatus: null }
ttl<60  → { error: "Error: KV_ERROR: expirationTtl is at least 60 seconds" }
the namespace's cell: scope=__KvNamespace:0b1fac48d6ac3067d910e69997c756bd68df3551d75c150d2733dedbb0749289
```
Observations, against the article:

- `getWithMetadata`with- `cacheTtl: 3600`answers with- `cacheStatus: null`, and so does- `list`: the documented "no edge cache".
- `list`reports each key's absolute- `expiration`(unix seconds) and its- `metadata`; the key with no TTL has neither.
- The `Profile`cell reads the namespace through`this.env.SESSIONS`exactly as the Worker does, and its`visits`counter is its own: a second call with a bad token still increments it. Shared data in the namespace, per-entity data in the cell.
- A TTL under a minute is refused with a named error, `KV_ERROR: expirationTtl is at least 60 seconds`. That is Cloudflare's floor, enforced loudly.
- The last line is celld's own log: the namespace is a cell with a scope of `__KvNamespace:<hash>`, started on first use. That is the "one writer per namespace" rule made visible.

## Part 3—D1: a database that is a cell (Chapter 9, Step 07, Chapter 6 "D1 is a cell")

`"d1_databases": [{ "binding": "DB", "database_name": "ledger" }]`"A D1 database on celld is one cell holding one SQLite database", with fencing, replication, and durable acknowledgement included. One database, one writer; more capacity is more databases. The article draws the line for *what goes where* sharply: per-entity data belongs in the entity's cell (Lab 1's `Ledger` kept each account's entries in the account's own SQLite); D1 is for the genuinely shared tables. Result caps are 100,000 rows or 32 MiB per binding result.

Migrations are Wrangler's: `NNNN_description.sql` files in `migrations/`, applied in numeric order, one transaction per file. The docs give exactly one way to apply them: `celld d1 migrations apply ledger --bucket …` against a fleet. They do not apply on a `celld dev` deploy. The cell below ships the file, and the table is not there after the deploy; the node logs nothing about migrations, and the `d1_migrations` bookkeeping table never appears. `celld d1` needs a running fleet (it finds a node through the bucket's node leases), so it cannot be pointed at the local store either. celld's own `examples/d1` sidesteps the question by running `CREATE TABLE IF NOT EXISTS` from the Worker. The workaround here is honest about what it is: an admin route that runs SQL through `env.DB.exec()`, fed the *same* migration file, the local equivalent of `celld d1 execute ledger --file`. On a fleet, use the real command.

```
const initSql = `CREATE TABLE IF NOT EXISTS entries (
  id INTEGER PRIMARY KEY,
  account TEXT NOT NULL,
  amount INTEGER NOT NULL,
  at INTEGER NOT NULL
);
`;
const d1Worker = {
  async fetch(request: Request, env: Env): Promise<Response> {
    const url = new URL(request.url);
    try {
      switch (`${request.method} ${url.pathname}`) {
        case "POST /admin/sql":   // local stand-in for `celld d1 execute --file`
          return Response.json(await env.DB.exec(await request.text()));
        case "GET /tables": {
          const { results } = await env.DB.prepare("SELECT name FROM sqlite_schema WHERE type = 'table' AND name NOT LIKE '\\_%' ESCAPE '\\'").all();
          return Response.json(results.map((r) => r.name));
        }
        case "POST /entries": {
          const { account, amount } = await request.json() as { account: string; amount: number };
          const r = await env.DB.prepare("INSERT INTO entries (account, amount, at) VALUES (?, ?, ?)").bind(account, amount, Date.now()).run();
          return Response.json(r.meta);
        }
        case "GET /balance":
          return Response.json(await env.DB
            .prepare("SELECT account, SUM(amount) AS balance FROM entries WHERE account = ? GROUP BY account")
            .bind(url.searchParams.get("account")).first());
        case "GET /summary": {
          const [count, byAccount] = await env.DB.batch([
            env.DB.prepare("SELECT COUNT(*) AS n FROM entries"),
            env.DB.prepare("SELECT account, SUM(amount) AS balance FROM entries GROUP BY account ORDER BY account"),
          ]);
          return Response.json({ count: count.results[0], byAccount: byAccount.results });
        }
        case "GET /dump":
          return Response.json({ bytes: (await env.DB.dump()).byteLength });
      }
      return new Response("not found", { status: 404 });
    } catch (e) {
      return Response.json({ error: String(e) }, { status: 500 });
    }
  },
};
await app.ship({ classes: [Profile], worker: d1Worker, wrangler, files: { "migrations/0001_init.sql": initSql } });
console.log("tables after deploy:", await app.json("/tables"));
console.log("log lines mentioning migrations:", JSON.stringify(app.logs({ grep: /migrat/i })));
// Apply the same file through the Worker — the local stand-in for `celld d1 execute ledger --file migrations/0001_init.sql`.
console.log("exec →", await app.json("/admin/sql", { method: "POST", body: await Deno.readTextFile(`${app.dir}/migrations/0001_init.sql`) }));
console.log("tables now:", await app.json("/tables"));
```
`const initSql = `CREATE TABLE IF NOT EXISTS entries (`52 lines```
reloaded bindings (2366 bytes)
tables after deploy: []
log lines mentioning migrations: ""
exec → { count: 1, duration: 0.1346 }
tables now: [ "entries" ]
```
With the table in place, the prepared-statement API from the article: `prepare().bind().run()` for writes, `.first()` for one row, `.all()` for many, and `batch()` for several statements in one round trip.

```
const entry = (account: string, amount: number) =>
  app.json("/entries", { method: "POST", body: JSON.stringify({ account, amount }) });
console.log("run   →", await entry("ada", 120));
await entry("ada", -45);
await entry("bob", 30);
console.log("first →", await app.json("/balance?account=ada"));
console.log("batch →", await app.json("/summary"));
console.log("dump  →", await app.json("/dump"));
console.log("bad   →", await app.json("/admin/sql", { method: "POST", body: "SELECT * FROM nowhere;" }));
```
`const entry = (account: string, amount: number) =>`10 lines```
run   → {
  changed_db: true,
  changes: 1,
  duration: 0.041203,
  last_row_id: 1,
  rows_read: 0,
  rows_written: 1,
  served_by: "celld",
  served_by_primary: true,
  served_by_region: "local",
  size_after: 53248
}
first → { account: "ada", balance: 75 }
batch → {
  count: { n: 3 },
  byAccount: [ { account: "ada", balance: 75 }, { account: "bob", balance: 30 } ]
}
dump  → {
  error: "Error: D1_ERROR: dump() is not implemented in celld; the database is a SQLite file in your own bucket"
}
bad   → { error: "Error: D1_EXEC_ERROR: no such table: nowhere" }
```
`run()`'s `meta` says who served the query: `served_by: "celld"`, `served_by_primary: true`, `served_by_region: "local"`, with `rows_written`, `last_row_id` and `size_after`. It is the same shape Cloudflare returns, filled in by the cell. `batch()` returns one result per statement. `dump()` is one of the listed gaps (with Time Travel and the D1 REST API), and celld says so in its own words: *"dump() is not implemented in celld; the database is a SQLite file in your own bucket."* Errors carry SQLite's message behind a named prefix: `D1_ERROR:` from a prepared statement, `D1_EXEC_ERROR:` from `exec()`.

## Part 4—R2: the fleet bucket wearing the R2 API (Chapter 9, Step 07, Chapter 6 "R2")

`"r2_buckets": [{ "binding": "FILES", "bucket_name": "files" }]`"An R2 binding stores its objects in the fleet bucket under `r2/<bucket_name>/`, which is the fleet bucket earning its name." So unlike KV, D1, and Queues, an R2 binding is *not* a cell: it is the object store itself, which is why the article recommends it for step results and artifacts that outgrow the 1 MiB Workflow caps. The gaps are `ssecKey` and `jurisdiction`, conditional writes on streamed bodies above 8 MiB, and multipart uploads that cannot resume on another node.

The article records one documented difference: "An object's `version` equals its content ETag, so identical content produces the same version." The cell below tests exactly that by writing the same bytes under two keys.

```
const r2Worker = {
  async fetch(request: Request, env: Env): Promise<Response> {
    const url = new URL(request.url);
    const key = url.searchParams.get("key") ?? "";
    const describe = (o: R2Object | null) => o && {
      key: o.key, size: o.size, version: o.version, etag: o.etag, httpEtag: o.httpEtag,
      checksums: o.checksums.toJSON(), contentType: o.httpMetadata?.contentType, customMetadata: o.customMetadata,
    };
    try {
      switch (`${request.method} ${url.pathname}`) {
        case "PUT /object":
          return Response.json(describe(await env.FILES.put(key, await request.text(), {
            httpMetadata: { contentType: request.headers.get("content-type") ?? "application/octet-stream" },
            customMetadata: { by: "notebook" },
          })));
        case "GET /object": {
          const o = await env.FILES.get(key);
          if (!o) return new Response("no such object", { status: 404 });
          return new Response(await o.text(), { headers: { etag: o.httpEtag, "content-type": o.httpMetadata?.contentType ?? "application/octet-stream" } });
        }
        case "GET /head":
          return Response.json(describe(await env.FILES.head(key)));
        case "GET /list": {
          const l = await env.FILES.list({ prefix: url.searchParams.get("prefix") ?? "", include: ["httpMetadata", "customMetadata"] });
          return Response.json({ truncated: l.truncated, objects: l.objects.map(describe) });
        }
        case "DELETE /object":
          await env.FILES.delete(key);
          return Response.json({ deleted: key });
      }
      return new Response("not found", { status: 404 });
    } catch (e) {
      return Response.json({ error: String(e) }, { status: 500 });
    }
  },
};
await app.ship({ classes: [Profile], worker: r2Worker, wrangler });
const put = (key: string, body: string) =>
  app.json(`/object?key=${key}`, { method: "PUT", body, headers: { "content-type": "application/json" } });
console.log("a →", await put("reports/a.json", '{"total":1}'));
console.log("b →", await put("reports/b.json", '{"total":1}'));   // the same bytes as a
console.log("c →", await put("reports/c.json", '{"total":2}'));
console.log("get  →", await app.text("/object?key=reports/a.json"));
console.log("head →", await app.json("/head?key=reports/c.json"));
console.log("list →", (await app.json<{ objects: { key: string; version: string }[] }>("/list?prefix=reports/")).objects.map((o) => `${o.key} @ ${o.version}`));
console.log("delete →", await app.json("/object?key=reports/c.json", { method: "DELETE" }), "→", await app.text("/object?key=reports/c.json"));
```
`const r2Worker = {`47 lines```
reloaded bindings (2674 bytes)
a → {
  key: "reports/a.json",
  size: 11,
  version: "74",
  etag: "74",
  httpEtag: '"74"',
  checksums: { md5: "fd74ebebe7b881ee7cd76d1dddd15ad4" },
  contentType: "application/json",
  customMetadata: { by: "notebook" }
}
b → {
  key: "reports/b.json",
  size: 11,
  version: "75",
  etag: "75",
  httpEtag: '"75"',
  checksums: { md5: "fd74ebebe7b881ee7cd76d1dddd15ad4" },
  contentType: "application/json",
  customMetadata: { by: "notebook" }
}
c → {
  key: "reports/c.json",
  size: 11,
  version: "76",
  etag: "76",
  httpEtag: '"76"',
  checksums: { md5: "ed3df89f2a272f12890cfe262cf61559" },
  contentType: "application/json",
  customMetadata: { by: "notebook" }
}
get  → {"total":1}
head → {
  key: "reports/c.json",
  size: 11,
  version: "76",
  etag: "76",
  httpEtag: '"76"',
  checksums: { md5: "ed3df89f2a272f12890cfe262cf61559" },
  contentType: "application/json",
  customMetadata: { by: "notebook" }
}
list → [ "reports/a.json @ 74", "reports/b.json @ 75", "reports/c.json @ 76" ]
delete → { deleted: "reports/c.json" } → no such object
```
`a` and `b` hold identical bytes: their `checksums.md5` agree, but their `version` / `etag` do **not**. On the local store each write gets the next number from a store-wide counter. So the docs' "identical content produces the same version" is not what `celld dev` does, as Chapter 6 and Chapter 9, Step 07 now note. The claim is about the fleet bucket, where an S3-style ETag of a single-part upload *is* the content hash; the local store is not S3, and it hands out its own ETags. Do not build on `version` equality for de-duplication unless you have checked it on the store you deploy to; `checksums.md5` is the content hash on both.

The `r2/<bucket_name>/` layout is checkable. `celld dev`'s local store, which stands in for the fleet bucket, is `<project>/.celld/dev/objects.sqlite3`, an `objects(key, body, etag, …)` table. Read it from the kernel (read-only, while the node is running):

```
const store = new DatabaseSync(`${app.dir}/.celld/dev/objects.sqlite3`, { readOnly: true });
console.table(store.prepare("SELECT key, etag, length(body) AS bytes FROM objects WHERE key LIKE 'r2/%' ORDER BY key").all().map((r) => ({ ...r })));
store.close();
```
`const store = new DatabaseSync(`${app.dir}/.celld/dev/objects.sqlite3`, { readOnly: tr …`3 lines┌───────┬───────────────────────────┬──────┬───────┐ │ (idx) │ key │ etag │ bytes │ ├───────┼───────────────────────────┼──────┼───────┤ │ 0 │ "r2/files/reports/a.json" │ 74 │ 11 │ │ 1 │ "r2/files/reports/b.json" │ 75 │ 11 │ └───────┴───────────────────────────┴──────┴───────┘

Two objects under `r2/files/…`, `c.json` gone, and the `etag` column is the same counter the binding reported as `version`. The R2 binding is a thin API over the store's own rows.

## Part 5—Queues: one writer, one consumer, four days (Chapter 9, Step 05, Chapter 6 "Queues")

```
"queues": {
  "producers": [{ "binding": "OUTBOX", "queue": "outbox" }],
  "consumers": [{ "queue": "outbox", "max_batch_size": 32 }]
}
```
Producer side: `env.OUTBOX.send(body)` and `sendBatch([{ body }, …])` from the Worker or a cell. Consumer side: a `queue(batch, env)` handler that `ack()`s or `retry()`s each message; a retried message becomes visible again after `delaySeconds` from `retry()` or `retryAll()`, defaulting to the consumer's `retry_delay` (celld adds no exponential backoff; compute one from `message.attempts` if you want it), and the queue keeps messages for four days. The constraints, all fail-loudly: one writer per queue (scale with more queues; past 256 concurrent producer calls the broker refuses with `cell overload: admission refused`), retention of four days and not configurable, no pull consumers or Queues HTTP API, and one consumer script per queue (a deployment in which two scripts consume one queue fails).

The consumer script may also export `fetch()`. The queues service page walks a producer, a consumer with retries, and a dead-letter queue, and celld's `examples/queues` exports `fetch` and `queue` from one script. The cell below uses that shape, a `queue()` handler on the same default export as `fetch()`, with a batch size of 5 and a 1-second batch timeout so the batching is visible, and one poison message that asks for a retry on its first attempt. The consumer records each batch in KV so the HTTP side can show it.

```
const queueWorker = {
  async fetch(request: Request, env: Env): Promise<Response> {
    const url = new URL(request.url);
    switch (url.pathname) {
      case "/send": {
        const n = Number(url.searchParams.get("n") ?? 1);
        await env.OUTBOX.send({ kind: "email", to: "ada@example.com", poison: url.searchParams.get("poison") === "1" });
        if (n > 1) await env.OUTBOX.sendBatch(Array.from({ length: n - 1 }, (_, i) => ({ body: { kind: "email", to: `user${i}@example.com` } })));
        return Response.json({ sent: n });
      }
      case "/delivered":
        return Response.json((await env.SESSIONS.get("outbox:batches", "json")) ?? []);
    }
    return new Response("not found", { status: 404 });
  },
  async queue(batch: MessageBatch, env: Env): Promise<void> {
    const log = ((await env.SESSIONS.get("outbox:batches", "json")) ?? []) as unknown[];
    log.push({ queue: batch.queue, size: batch.messages.length,
      messages: batch.messages.map((m) => `${m.body.to} (attempt ${m.attempts}${m.body.poison ? ", poison" : ""})`) });
    await env.SESSIONS.put("outbox:batches", JSON.stringify(log));
    for (const msg of batch.messages) {
      if (msg.body.poison && msg.attempts < 2) { msg.retry({ delaySeconds: 1 }); continue; }   // redelivered after delaySeconds
      msg.ack();
    }
  },
};
wrangler.queues = {
  producers: [{ binding: "OUTBOX", queue: "outbox" }],
  consumers: [{ queue: "outbox", max_batch_size: 5, max_batch_timeout: 1 }],
};
app.clearLogs();
await app.ship({ classes: [Profile], worker: queueWorker, wrangler });
console.log(app.logs({ grep: /^env\.OUTBOX/, last: 1 }));
console.log(await app.json("/send?n=7&poison=1"));
// With a consumer attached, every reload is a full restart and the port blinks for a few seconds — poll, don't sleep.
let batches: unknown[] = [];
for (let i = 0; i < 60 && batches.length < 3; i++) {
  await sleep(500);
  batches = await app.json<unknown[]>("/delivered").catch(() => batches);
}
for (const b of batches) console.log(JSON.stringify(b));
console.log("restarts during this cell:", app.logs({ grep: /restarting the application/ }).split("\n").filter(Boolean).length);
console.log(app.logs({ grep: /cell_isolate_startup_timing.*__Queue/, last: 1 }).replace(/^.*scope=/, "the queue's cell: scope=").replace(/ node=.*$/, ""));
```
`const queueWorker = {`46 lines```
reloaded bindings (2022 bytes)
env.OUTBOX (Queue)         outbox
{ sent: 7 }
{"queue":"outbox","size":5,"messages":["ada@example.com (attempt 1, poison)","user0@example.com (attempt 1)","user1@example.com (attempt 1)","user2@example.com (attempt 1)","user3@example.com (attempt 1)"]}
{"queue":"outbox","size":2,"messages":["user4@example.com (attempt 1)","user5@example.com (attempt 1)"]}
```
It works. Attaching a consumer changes celld's reload path: from then on every reload is a full restart of the node, a few seconds during which the port is closed, hence the polling loop. Seven messages arrive as a batch of 5 and a batch of 2 (`max_batch_size`, with `max_batch_timeout` closing the short one). The poison message, retried with `delaySeconds: 1`, comes back in a third batch of its own; the restart cell below counts all three. Every message was handled by the same script that serves HTTP, the documented shape. The consumer is attached to *this* script: celld's binary carries the message "queue consumer declares `script_name`; celld attaches it to the current script", so a `script_name` pointing elsewhere is ignored rather than honored, and there is no second entry point to attach it to. Keeping ingress and consumption separable is sound design, not a rule.

The queue itself is a cell, like the namespace: it appears as a `__Queue` cell in the local store, as the end of this Part shows.

Now the fail-loudly side. The consumer is declared; remove the handler and leave the declaration, which is the mistake the article's warning is really about. Write `src/index.js` by hand, as in Part 1:

```
app.clearLogs();
await Deno.writeTextFile(`${app.dir}/src/index.js`, "export default { fetch() { return new Response('ingress only'); } };\n");
await sleep(4000);
console.log(app.logs({ grep: /restarting|Error|Caused|queue consumer|exited/ }));
console.log("node reachable?", await app.fetch("/delivered").then((r) => r.status, (e) => String(e).split("\n")[0]));
```
`app.clearLogs();`5 lines```
  ● restarting the application
Error: stateless Worker failed to load
Caused by:
    queue consumer has no queue handler
Error: the local node exited before it announced its operator listener
node reachable? TypeError: fetch failed
```
This one does not end as a failed reload. Because the consumer attachment changed, celld took the *restart* path ("restarting the application", an exact-generation reload), the new Worker failed to load with the named error `queue consumer has no queue handler`, and the local node **exited**. Nothing is serving on the port any more. That is the strongest form of fail-loudly: a consumer shape celld does not model never runs silently, but on `celld dev` it also takes the node down with it, and a `ship()` at this point would wait for a reload that never comes. Recovery is a stop, a good ship, and a start, without `--clean`, so the store is kept.

```
await app.stop();
await app.ship({ classes: [Profile], worker: queueWorker, wrangler });
await app.start({ logs: true });
console.log("delivery log survived the restart:", (await app.json<unknown[]>("/delivered")).length, "batches");
const store2 = new DatabaseSync(`${app.dir}/.celld/dev/objects.sqlite3`, { readOnly: true });
console.log("cell kinds in the local store:",
  store2.prepare("SELECT DISTINCT substr(key, 7, instr(substr(key, 7), ':') - 1) AS kind FROM objects WHERE key LIKE 'cells/%'").all().map((r) => r.kind));
store2.close();
```
`await app.stop();`9 linescelld dev stopped (exit 1) celld dev ready at http://127.0.0.1:9902 (project /home/jovyan/.cache/celld-nb/apps/bindings) delivery log survived the restart: 3 batches cell kinds in the local store: [ "Profile", "__D1Database", "__KvNamespace", "__Queue" ]

The last line is the theme of this notebook, read straight off the store: alongside the `Profile` cells sit `__KvNamespace`, `__D1Database`, and `__Queue` cells, one per namespace, database, and queue, and the R2 objects are the `r2/` rows from Part 4. Every cross-cutting service is a cell (or the bucket itself) underneath.

### Exercise—a session store

Sessions are the article's own KV example (`session:${token}` with `expirationTtl: 3600`), and an audit trail is a genuinely shared table: D1's job, not a per-user cell's. Complete `sessionWorker`:

- `POST /login?user=<name>`: mint a token (- `crypto.randomUUID()`), write- `session:<token>`→- `{ "user": <name> }`to- `SESSIONS`with a- **one-hour TTL**, insert a row- `(user, event = 'login', at)`into the- `audit`table, and answer- `{ "token": … }`.
- `GET /whoami`: read the- `x-session`header, look the token up in- `SESSIONS`; answer- `{ "user": … }`, or- `401`when there is no such session.
- `GET /audit`: the- `audit`rows, oldest first, as- `[{ user, event, at }, …]`.

The `/admin/sql` route is already written (the checker uses it to apply `migrations/0002_audit.sql`, the way Part 3 did), and so is the error wrapper. `sessionWorker` has no `queue()` handler, so the checker first drops the consumer declaration from Part 5; shipping without doing that would take the node down, as Part 5 showed. There is no in-kernel fake for KV or D1, so the checker runs on celld only; it prints ✗ until the TODOs are filled in.

**Hints**, from a nudge to a near-solution. Try the exercise first, then read only as far as you need.

- **Concept.**Two stores, two jobs. A session is per-token, short-lived state: KV with a TTL. The audit trail is one shared table: D1.- `/login`writes both;- `/whoami`reads only KV.
- **API.**- `env.SESSIONS.put(key, JSON.stringify({ user }), { expirationTtl: 3600 })`writes a session, and- `env.SESSIONS.get(key, "json")`returns the object or- `null`.- `env.DB.prepare(sql).bind(…).run()`inserts a row;- `.all()`returns- `{ results }`.
- **Sketch.**- `/login`: mint the token, write the session, insert- `(user, 'login', Date.now())`into- `audit`, answer- `{ token }`.- `/whoami`: a found session answers- `{ user }`, a missing one answers 401.- `/audit`: answer the- `results`of the- `ORDER BY id`query as they come back.

Full worked solutions are intentionally not included. The checker cell is the verification: every line turns from ✗ to ✓ once the implementation is right.

```
const auditSql = "CREATE TABLE IF NOT EXISTS audit (id INTEGER PRIMARY KEY, user TEXT NOT NULL, event TEXT NOT NULL, at INTEGER NOT NULL);\n";
const sessionWorker = {
  async fetch(request: Request, env: Env): Promise<Response> {
    const url = new URL(request.url);
    try {
      switch (`${request.method} ${url.pathname}`) {
        case "POST /admin/sql":
          return Response.json(await env.DB.exec(await request.text()));
        case "POST /login": {
          const user = url.searchParams.get("user") ?? "";
          // TODO: token = crypto.randomUUID(); SESSIONS.put(`session:${token}`, …, { expirationTtl: 3600 });
          //       DB.prepare("INSERT INTO audit …").bind(user, "login", Date.now()).run(); answer { token }
          return new Response("not implemented", { status: 501 });
        }
        case "GET /whoami": {
          const token = request.headers.get("x-session") ?? "";
          // TODO: SESSIONS.get(`session:${token}`, "json") → { user } or 401
          return new Response("not implemented", { status: 501 });
        }
        case "GET /audit":
          // TODO: DB.prepare("SELECT user, event, at FROM audit ORDER BY id").all() → results
          return new Response("not implemented", { status: 501 });
      }
      return new Response("not found", { status: 404 });
    } catch (e) {
      return Response.json({ error: String(e) }, { status: 500 });
    }
  },
};
```
`const auditSql = "CREATE TABLE IF NOT EXISTS audit (id INTEGER PRIMARY KEY, user TEXT  …`30 lines```
// ── checker (celld only: these bindings have no kernel-side fake) ──────────
wrangler.queues = { producers: [{ binding: "OUTBOX", queue: "outbox" }] };   // no queue() handler here → no consumer (Part 5)
await app.ship({ classes: [Profile], worker: sessionWorker, wrangler, files: { "migrations/0002_audit.sql": auditSql } });
await app.json("/admin/sql", { method: "POST", body: await Deno.readTextFile(`${app.dir}/migrations/0002_audit.sql`) });
const check = (label: string, ok: boolean, detail: unknown) => console.log(ok ? "✓" : "✗", label, ok ? "" : `— got ${JSON.stringify(detail)}`);
const login = await app.fetch("/login?user=ada", { method: "POST" });
const { token } = (await login.json().catch(() => ({}))) as { token?: string };
check("POST /login answers 200 with a token", login.status === 200 && typeof token === "string" && token.length > 0, login.status);
const who = await app.fetch("/whoami", { headers: { "x-session": token ?? "" } });
const whoBody = await who.json().catch(() => null) as { user?: string } | null;
check("GET /whoami resolves the token to ada", who.status === 200 && whoBody?.user === "ada", [who.status, whoBody]);
const bad = await app.fetch("/whoami", { headers: { "x-session": "not-a-token" } });
check("GET /whoami with an unknown token is 401", bad.status === 401, bad.status);
const audit = await app.fetch("/audit");
const rows = (await audit.json().catch(() => [])) as { user: string; event: string; at: number }[];
check("GET /audit has ada's login row", audit.status === 200 && rows.some((r) => r.user === "ada" && r.event === "login" && typeof r.at === "number"), [audit.status, rows]);
```
`wrangler.queues = { producers: [{ binding: "OUTBOX", queue: "outbox" }] };   // no que …`21 linesreloaded bindings (1942 bytes) ✗ POST /login answers 200 with a token — got 501 ✗ GET /whoami resolves the token to ada — got [501,null] ✗ GET /whoami with an unknown token is 401 — got 501 ✗ GET /audit has ada's login row — got [501,[]]

## Clean up

Stop the node. State stays in the project directory's `.celld/dev`; the next `app.start()` without `clean: true` resumes it, with the sessions, the ledger, the objects, and the queue's four-day backlog included.

`await stopAll();``await stopAll();`1 linecelld dev stopped (exit 0)

## Where next

That is Steps 05 and 07 of the guide and the matching subsections of Chapter 6: four services with Cloudflare's APIs, each a cell (or the bucket) underneath, one writer each, declared in a configuration celld validates loudly. Two places where `celld dev` behaves differently from a fleet, both observed above, both documented in the articles, and worth re-checking on the release you deploy, and one documented shape confirmed above:

- **D1 migrations do not apply on**Apply the file yourself locally; use- `celld dev`'s deploy.- `celld d1 migrations apply`on a fleet.
- **R2**- `version`on the local store is a write counter, not a content hash.- `checksums.md5`is the content hash everywhere.
- **A**, and a declared consumer without a handler takes the node down.- `queue()`handler beside- `fetch()`is the documented shape
- **Lab 1—The cell model**is the foundation: cells, addressing, one thread per cell, SQL storage, alarms, the dev loop.
- **Lab 3—Processes and lifecycle**covers Workflows (Step 06), WebSockets and hibernation, cron triggers, RPC, and reading celld's lifecycle events.

Left to the articles: the fleet bucket that makes R2 "earn its name", queue overload (`503`, `Retry-After: 1`, `X-Celld-Overload: cell`), the `celld kv` / `celld queue` / `celld d1` operator commands. All of these need a fleet, and `celld dev`'s local store is not one.

**Maintenance.** This export is frozen at celld v0.6.0, executed 2026·09·26. Re-execute it on each new celld release and note what changed: upstream moves fast, with five releases (v0.4.0 through v0.6.0) between 2026·08·28 and 2026·09·26.

This lab draws on: celld: documentation at v0.6.0 (3 entries). The full entries are in the Bibliography.


---

<!-- https://tlockney.github.io/celld-book/lab-3.html -->

This chapter is an executed notebook: every output below came from a run against celld v0.6.0 on 2026·09·26. To run it yourself, download this notebook and the helper celld_nb.ts into the same folder (the setup cell imports it), or get all three labs and the helper as a zip. Open the notebook in JupyterLab with a Deno kernel; the setup cell installs celld and esbuild if they are missing.

**Written against celld v0.6.0.** Appendix B defines the terms it uses. This notebook covers Chapter 2 "Cell states", Chapter 6 "Runtime API surface", "Workflows", and "Cron, alarms, and WebAssembly", and Chapter 7 "Designing applications around cells", together with Steps 04 and 06 of the guide: JS RPC, hibernatable WebSockets and what a wake actually is, Workflows and the replay rule, cron triggers, and the entity/process distinction that decides which primitive you reach for.

As in Lab 1, this notebook is a **driver** for a real `celld dev` process. Cells define Durable Object and Workflow classes as executable TypeScript against the kernel-side shims in `celld_nb.ts` (`DurableObject`, `WorkflowEntrypoint`, `fakeCtx()`) and run them in the kernel where that teaches something. Then `app.ship()` serializes them with `Function.prototype.toString()` into `src/index.js` and celld hot-reloads. Shipped classes must be self-contained, and what celld runs is the JavaScript left after the kernel strips the types. This notebook adds one thing Lab 1 did not need: the Deno kernel has a native `WebSocket` client, so cells here connect to a cell's WebSockets directly.

## Setup

Two start options matter here. `logs: true` passes `--logs`, so the node's INFO lines land in the kernel; those are the lifecycle events this notebook reads with `app.logs()`. `CELLD_IDLE_EVICT_S` is the opt-in from Chapter 2; it *"sets how many seconds without work send an idle resident cell to hibernation, and when it is unset only memory pressure or the residency cap removes one."* Two seconds makes hibernation something you can watch.

```
import { App, DurableObject, WorkflowEntrypoint, fakeCtx, stopAll, sleep, type DurableObjectNamespace, type FakeContext } from "./celld_nb.ts";
const app = new App("processes", 9903);
console.log("project dir:", app.dir);
```
`import { App, DurableObject, WorkflowEntrypoint, fakeCtx, stopAll, sleep, type Durable …`4 linesproject dir: /home/jovyan/.cache/celld-nb/apps/processes

## Part 1—JS RPC (Chapter 6)

Chapter 6's runtime table lists JS RPC as **Yes**: a Worker calls methods on a Durable Object stub instead of wrapping every operation in a `Request`. The same line carries three caveats: *"A stub cannot cross an isolate boundary; an  AbortSignal passes through on the same node but not across a node boundary; retries only when the peer attempt provably did not start."* In short: call methods freely, do not hand a stub to another cell, and design the methods so a retried call is harmless.

`Tally` keeps named counters in key-value storage and exposes three public methods. There is no `fetch()` at all; every public method on the class is callable through the stub. `Tally` is also the notebook's side channel: later Parts have Workflows and the cron handler bump counters in it, so their progress is visible from the kernel.

The class runs in the kernel first, where "RPC" is just calling the methods.

```
interface Env {
  TALLY: DurableObjectNamespace;
}
class Tally extends DurableObject<Env> {
  async increment(key: string, by = 1): Promise<number> {
    const n = ((await this.ctx.storage.get<number>(key)) ?? 0) + by;
    await this.ctx.storage.put(key, n);
    return n;
  }
  async read(key: string): Promise<number> {
    return (await this.ctx.storage.get<number>(key)) ?? 0;
  }
  async all(): Promise<Record<string, number>> {
    return Object.fromEntries(await this.ctx.storage.list<number>());
  }
}
const tally = new Tally(fakeCtx("kernel"), {} as Env);
console.log(await tally.increment("visits"), await tally.increment("visits"), await tally.increment("errors", 3));
console.log(await tally.all());
```
`interface Env {`21 lines```
1 2 3
{ errors: 3, visits: 2 }
```
The Worker below is the router the whole notebook grows around. Its `/tally/<key>` routes get a stub with `env.TALLY.get(id)` and call `increment` / `read` / `all` on it. No `Request` is constructed, and the return values come back as ordinary promises. Everything else falls through to a generic `/do/<BINDING>/<name>/<path>` passthrough for the cells later Parts add, so the router does not have to change every time.

```
const worker = {
  async fetch(request: Request, env: Record<string, DurableObjectNamespace>): Promise<Response> {
    const url = new URL(request.url);
    const [, head, ...rest] = url.pathname.split("/");
    if (head === "tally") {
      const tally = env.TALLY.get(env.TALLY.idFromName("main"));     // a stub: methods, not fetch()
      const key = rest[0];
      if (!key) return Response.json(await tally.all());
      if (request.method === "POST") return Response.json({ [key]: await tally.increment(key) });
      return Response.json({ [key]: await tally.read(key) });
    }
    if (head === "do") {
      const [binding, name, ...path] = rest;
      const ns = env[binding];
      if (!ns) return new Response(`no binding ${binding}`, { status: 404 });
      const cellUrl = new URL(`http://cell/${path.join("/")}${url.search}`);
      return ns.get(ns.idFromName(name)).fetch(new Request(cellUrl, request));
    }
    return new Response("GET|POST /tally/<key> · GET /tally · /do/<BINDING>/<name>/<path>", { status: 404 });
  },
};
const wrangler = {
  durable_objects: { bindings: [{ name: "TALLY", class_name: "Tally" }] },
  migrations: [{ tag: "v1", new_sqlite_classes: ["Tally"] }],
};
await app.ship({ classes: [Tally], worker, wrangler });
await app.start({ clean: true, logs: true, env: { CELLD_IDLE_EVICT_S: "2" } });
console.log(await app.json("/tally/visits", { method: "POST" }));
console.log(await app.json("/tally/visits", { method: "POST" }));
console.log(await app.json("/tally/errors", { method: "POST" }));
console.log(await app.json("/tally"));
```
`const worker = {`34 lines```
celld  /home/jovyan/.cache/celld-nb/tools/bin/celld  (celld 0.6.0)
esbuild /home/jovyan/.cache/celld-nb/tools/esbuild/node_modules/.bin/esbuild
celld dev ready at http://127.0.0.1:9903  (project /home/jovyan/.cache/celld-nb/apps/processes)
{ visits: 1 }
{ visits: 2 }
{ errors: 1 }
{ errors: 1, visits: 2 }
```
Same class, same answers as the in-kernel run. Notice what `wrangler.jsonc` does not say: nothing declares `Tally` an RPC class. Any public method on a `DurableObject` subclass is reachable through the stub; `fetch()` is just the method a `Request` is routed to.

## Part 2—WebSockets and hibernation (Step 04, Chapter 2 "Cell states", Chapter 7 "A working cell")

Chapter 7 calls the hibernating chat room *"the whole application shape"*, and Step 04 shows it with SQL state added. The key line is `this.ctx.acceptWebSocket(server)`. Accepting the socket through the hibernation API rather than `addEventListener` is what *"lets celld evict an idle room from memory while every client stays connected. A thousand quiet rooms cost almost nothing; a message wakes the one room it addresses."*

`ChatRoom` is the article's class with two additions that make the lifecycle visible:

- A `console.log`in the constructor. celld forwards a cell's console output into the node log as`cell_console`INFO lines, so`app.logs()`shows every time the constructor runs.
- `#firstEvent`, an in-memory flag that starts- `true`in every new instance. The first handler to run on a fresh instance bumps a- `wakes`counter in storage. Memory holds nothing across a wake, so- `wakes`counts exactly how many times celld has constructed this room, and it does so without any work in the constructor, which Step 04 says to keep trivial.

The kernel has no `WebSocketPair`, so the upgrade path cannot run here, but `webSocketMessage` can: give `fakeCtx()` a `getWebSockets()` that returns fake peers and watch the broadcast and the SQL log.

```
interface WsContext extends FakeContext {
  acceptWebSocket(ws: WebSocket): void;
  getWebSockets(): WebSocket[];
}
class ChatRoom extends DurableObject<Env> {
  #firstEvent = true;
  constructor(ctx: FakeContext, env: Env) {
    super(ctx, env);
    console.log(`ChatRoom constructor (${ctx.id.name ?? ctx.id})`);   // shows up in app.logs() as cell_console
  }
  async #onEvent(): Promise<void> {
    this.ctx.storage.sql.exec("CREATE TABLE IF NOT EXISTS log (at INTEGER, body TEXT)");
    if (!this.#firstEvent) return;
    this.#firstEvent = false;
    await this.ctx.storage.put("wakes", ((await this.ctx.storage.get<number>("wakes")) ?? 0) + 1);
  }
  async fetch(request: Request): Promise<Response> {
    await this.#onEvent();
    const ctx = this.ctx as unknown as WsContext;
    if (new URL(request.url).pathname === "/ws") {
      const pair = new WebSocketPair();
      const [client, server] = Object.values(pair);
      ctx.acceptWebSocket(server);                 // hibernatable — the room can sleep
      return new Response(null, { status: 101, webSocket: client });
    }
    return Response.json({
      sockets: ctx.getWebSockets().length,
      wakes: await this.ctx.storage.get<number>("wakes"),
      log: this.ctx.storage.sql.exec<{ body: string }>("SELECT body FROM log ORDER BY at").toArray().map((r) => r.body),
    });
  }
  async webSocketMessage(ws: WebSocket, message: string | ArrayBuffer): Promise<void> {
    await this.#onEvent();
    this.ctx.storage.sql.exec("INSERT INTO log (at, body) VALUES (?, ?)", Date.now(), String(message));
    for (const peer of (this.ctx as unknown as WsContext).getWebSockets()) {
      if (peer !== ws) peer.send(message);          // one room, no message bus
    }
  }
  async alarm(): Promise<void> {
    // prune history older than a day, then reschedule
    this.ctx.storage.sql.exec("DELETE FROM log WHERE at < ?", Date.now() - 86_400_000);
    await this.ctx.storage.setAlarm(Date.now() + 3_600_000);
  }
}
// In the kernel: fake peers, real SQL.
const received: string[] = [];
const peer = (name: string) => ({ send: (m: string | ArrayBuffer) => received.push(`${name} ← ${m}`) }) as unknown as WebSocket;
const [a, b, c] = [peer("a"), peer("b"), peer("c")];
const roomCtx = Object.assign(fakeCtx("lobby"), { acceptWebSocket() {}, getWebSockets: () => [a, b, c] });
const room = new ChatRoom(roomCtx, {} as Env);
await room.webSocketMessage(a, "hello from a");
await room.webSocketMessage(c, "hi from c");
console.log(received);
console.log(await (await room.fetch(new Request("http://cell/"))).json());
```
`interface WsContext extends FakeContext {`61 lines```
ChatRoom constructor (lobby)
[
  "b ← hello from a",
  "c ← hello from a",
  "a ← hi from c",
  "b ← hi from c"
]
{ sockets: 3, wakes: 1, log: [ "hello from a", "hi from c" ] }
```
Ship it, then open two real WebSocket clients from the kernel to the same room, send from one, and receive on the other. The room reports its `getWebSockets()` count over the passthrough route.

```
wrangler.durable_objects.bindings.push({ name: "ROOM", class_name: "ChatRoom" });
wrangler.migrations.push({ tag: "v2", new_sqlite_classes: ["ChatRoom"] });
await app.ship({ classes: [Tally, ChatRoom], worker, wrangler });
const connect = (name: string) => {
  const ws = new WebSocket(`ws://127.0.0.1:${app.port}/do/ROOM/lobby/ws`);
  ws.onmessage = (e) => console.log(`${name} received:`, e.data);
  ws.onclose = (e) => console.log(`${name} closed (code ${e.code})`);
  return new Promise<WebSocket>((resolve) => ws.onopen = () => resolve(ws));
};
const [alice, bob] = await Promise.all([connect("bob"), connect("alice")].reverse());
alice.send("hello from alice");
await sleep(200);
bob.send("hi alice");
await sleep(200);
console.log(await app.json("/do/ROOM/lobby/"));
```
`wrangler.durable_objects.bindings.push({ name: "ROOM", class_name: "ChatRoom" });`17 lines```
reloaded processes (3079 bytes)
bob received: hello from alice
alice received: hi alice
{ sockets: 2, wakes: 1, log: [ "hello from alice", "hi alice" ] }
```
Two sockets, one wake, both messages in the SQL log, and each message delivered to the other peer only. In the state diagram's terms the room is now *resident*: in memory, idle between messages.

### Hibernation, observed

Chapter 2's lifecycle diagram labels the hibernated state *"evicted, but still placed"*: *"WS clients stay connected"* and the cell *"stays on its node"*. On the next message, *"the constructor runs again"*. With `CELLD_IDLE_EVICT_S=2` the eviction should happen a few seconds after the last message. Clear the log, wait, and look for what celld actually writes. Then send a message through the still-open socket and look again.

```
const tidy = (line: string) => line
  .replace(/^\S+Z\s+INFO\s+/, "")                                   // timestamp + level
  .replace(/(ChatRoom|Tally)[:.]?[0-9a-f]{64}/g, "$1:…")               // the cell's 64-hex id
  .replace(/ (node|region|runtime_version|failure_phase)=\S+/g, "");   // node identity noise
const show = (grep: RegExp) => console.log(app.logs({ grep }).split("\n").filter(Boolean).map(tidy).join("\n"));
app.clearLogs();
await sleep(5000);
console.log("── while idle:");
show(/ChatRoom/);
console.log("\nsockets still open on the client side:", alice.readyState === WebSocket.OPEN, bob.readyState === WebSocket.OPEN);
console.log("\n── after a message to the hibernated room:");
app.clearLogs();
alice.send("anyone awake?");
await sleep(400);
show(/ChatRoom|cell_console/);
console.log("\nroom:", await app.json("/do/ROOM/lobby/"));
```
`const tidy = (line: string) => line`18 lines```
── while idle:
celld::ltx_repl: published authoritative handoff snapshot event="ltx_handoff_snapshot" cell="ChatRoom:…" epoch=1 max_txid=3 bytes=2513 elapsed_ms=64
celld::ltx_repl: final durability barrier passed event="eviction_durability_barrier" cell="ChatRoom:…" epoch=1 artifact=Snapshot elapsed_ms=65
sockets still open on the client side: true true
── after a message to the hibernated room:
bob received: anyone awake?
celld::ltx_repl: reused local eviction snapshot cell="ChatRoom:…" epoch=2
celld::runtime: cell isolate startup completed event="cell_isolate_startup_timing" outcome="ready" scope=ChatRoom:… epoch=2 fresh=false total_us=620
celld::runtime: cell runtime published event="cell_runtime_publication" outcome="published" scope=ChatRoom:… epoch=2 isolate_startup_us=620
cell_console: ChatRoom constructor (lobby)
room: {
  sockets: 2,
  wakes: 2,
  log: [ "hello from alice", "hi alice", "anyone awake?" ]
}
```
Read the two blocks against the diagram. While idle, celld published an *"authoritative handoff snapshot"* of the room's SQLite and passed a *"final durability barrier"* (`eviction_durability_barrier`): state made durable, memory freed. Both client sockets stayed open. Then one message did what the article says it does. The room came back (`cell_isolate_startup_timing` with `fresh=false`, restored from the eviction snapshot it had just written), the constructor's `console.log` appeared again, `wakes` went from 1 to 2, and the room still held both sockets: bob received the message with no reconnect.

That is the whole economics of the model in one screen, and the reason for three rules from Step 04 and Chapter 7:

- **Keep the constructor trivial.**- *"It runs on every wake, including each message to a hibernated cell. Restore from*- `storage`in the handler, not the constructor."- `ChatRoom`restores nothing in its constructor;- `#onEvent`does the work, lazily.
- **Batch WebSocket messages.**- *"Each frame costs a context switch; pack many small logical messages into one frame with an envelope format. Fewer, larger messages beat many small ones."*
- **Outbound WebSockets do not survive a move.**- *"An outbound socket keeps the cell resident and dies when the cell changes nodes. Keep connection intent in storage and re-dial after activation."*Inbound hibernatable sockets are the exception, as you just saw, but only across hibernation, not across a change of owner:- *"a transport cannot move to a new owner, so reconnect with a stable operation ID."*

One thing the articles do not mention that you will hit in the dev loop: a **hot reload is a change of deployment, not a hibernation**. If sockets are open when you `ship()`, celld drains the old deployment for up to 25 s (`celld preserve drain reached its 25000ms deadline … websockets=2`) and then closes them with an abnormal close. So close the clients before shipping again.

```
alice.close();
bob.close();
await sleep(300);
```
`alice.close();`3 linesalice closed (code 1000) bob closed (code 1000)

## Part 3—Workflows (Step 06, Chapter 6 "Workflows")

*"Cells model entities: named things whose state persists indefinitely. Workflows are the process primitive built on top of them: a sequence of steps that ends, with each step's result stored durably so the sequence survives crashes and restarts."* A workflow extends `WorkflowEntrypoint` and does its work in `run(event, step)`. `step.do(name, fn)` runs a callback and stores its result; `step.sleep(name, duration)` waits durably.

`ReportBuilder` is Step 06's example with the outbound `fetch` replaced by a computation on the params, so the notebook does not depend on the network, plus one addition: every stage bumps a counter in the `Tally` cell from Part 1 through its stub. The stages are the top of `run()`, each step's callback, and the code *outside* steps before and after the sleep. That trace is how the next cells make the replay rule visible.

First, in the kernel. `WorkflowStep` is not in `celld_nb.ts`, so a minimal fake is defined here: `do` runs the callback, `sleep` returns. The in-kernel `env` hands the workflow a real in-kernel `Tally` in place of the stub. The methods are the same, so the workflow cannot tell.

```
interface WorkflowEvent<P> { payload: P; instanceId: string; timestamp: Date }
interface WorkflowStep {
  do<T>(name: string, fn: () => Promise<T>): Promise<T>;
  sleep(name: string, duration: string | number): Promise<void>;
}
interface WorkflowInstance { id: string; status(): Promise<{ status: string; output?: unknown; error?: unknown }>; restart(): Promise<void> }
interface WorkflowBinding { create(opts?: { id?: string; params?: unknown }): Promise<WorkflowInstance>; get(id: string): Promise<WorkflowInstance> }
interface ReportParams { text: string }
interface WorkflowEnv extends Env { REPORTS: WorkflowBinding }
class ReportBuilder extends WorkflowEntrypoint<WorkflowEnv> {
  async run(event: WorkflowEvent<ReportParams>, step: WorkflowStep) {
    const tally = this.env.TALLY.get(this.env.TALLY.idFromName("main"));
    await tally.increment("wf:top of run()");                         // outside any step
    const summary = await step.do("summarize", async () => {
      await tally.increment("wf:step summarize");
      const words = event.payload.text.split(/\s+/).filter(Boolean);
      return { words: words.length, longest: words.reduce((a, b) => (b.length > a.length ? b : a), "") };
    });
    await tally.increment("wf:before sleep");                         // outside any step
    await step.sleep("cool off", "4 seconds");
    await tally.increment("wf:after sleep");                          // outside any step
    return await step.do("store summary", async () => {
      await tally.increment("wf:step store");                          // step.do callbacks are the only safe home for side effects
      return { ...summary, storedAt: Date.now() };
    });
  }
}
// A WorkflowStep for the kernel: no durability, just run the callbacks.
const fakeStep = (): WorkflowStep => ({
  async do(name, fn) { console.log(`  step.do "${name}"`); return await fn(); },
  async sleep(name, duration) { console.log(`  step.sleep "${name}" (${duration}) — skipped in the kernel`); },
});
const kernelTally = new Tally(fakeCtx("main"), {} as Env);
const kernelEnv = { TALLY: { idFromName: (n: string) => n, newUniqueId: () => "x", get: () => kernelTally } } as unknown as WorkflowEnv;
const report = new ReportBuilder(null, kernelEnv);
console.log(await report.run({ payload: { text: "the quick brown fox jumps over the lazy dog" }, instanceId: "k1", timestamp: new Date() }, fakeStep()));
console.log(await kernelTally.all());
```
`interface WorkflowEvent<P> { payload: P; instanceId: string; timestamp: Date }`40 linesstep.do "summarize" step.sleep "cool off" (4 seconds) — skipped in the kernel step.do "store summary"

Now on celld. The configuration is Step 06's `workflows` entry: `binding` is the name on `env`, `class_name` the export, `name` the workflow's fleet-wide identity. The router gains Step 06's two routes, `/create` → `env.REPORTS.create({ params })` and `/status?id=`, plus `/restart?id=` for later. Poll the status from the kernel until the instance completes.

```
const workflowRouter = {
  async fetch(request: Request, env: WorkflowEnv & Record<string, DurableObjectNamespace>): Promise<Response> {
    const url = new URL(request.url);
    if (url.pathname === "/create") {
      const instance = await env.REPORTS.create({ params: { text: url.searchParams.get("text") ?? "" } });
      return Response.json({ id: instance.id });
    }
    const id = url.searchParams.get("id");
    if (url.pathname === "/status" && id) return Response.json(await (await env.REPORTS.get(id)).status());
    if (url.pathname === "/restart" && id) {
      const instance = await env.REPORTS.get(id);
      await instance.restart();
      return Response.json(await instance.status());
    }
    // everything else: the Part 1 router (tally + /do passthrough)
    const [, head, ...rest] = url.pathname.split("/");
    if (head === "tally") {
      const tally = env.TALLY.get(env.TALLY.idFromName("main"));
      const key = rest[0];
      if (!key) return Response.json(await tally.all());
      if (request.method === "POST") return Response.json({ [key]: await tally.increment(key) });
      return Response.json({ [key]: await tally.read(key) });
    }
    if (head === "do") {
      const [binding, name, ...path] = rest;
      const ns = env[binding];
      if (!ns) return new Response(`no binding ${binding}`, { status: 404 });
      return ns.get(ns.idFromName(name)).fetch(new Request(new URL(`http://cell/${path.join("/")}${url.search}`), request));
    }
    return new Response("/create?text= · /status?id= · /restart?id= · /tally · /do/…", { status: 404 });
  },
};
Object.assign(wrangler, { workflows: [{ binding: "REPORTS", name: "report-builder", class_name: "ReportBuilder" }] });
const spec = { classes: [Tally, ChatRoom, ReportBuilder], worker: workflowRouter, wrangler };
await app.ship(spec);
type Status = { status: string; output?: unknown };
const watch = async (id: string, maxMs = 15_000) => {
  const seen: string[] = [];
  const t0 = Date.now();
  while (Date.now() - t0 < maxMs) {
    const s = await app.json<Status>(`/status?id=${id}`);
    if (seen.at(-1) !== s.status) { seen.push(s.status); console.log(`  +${((Date.now() - t0) / 1000).toFixed(1)}s  ${s.status}`); }
    if (s.status === "complete" || s.status === "errored") return s;
    await sleep(250);
  }
  throw new Error(`still ${seen.at(-1)} after ${maxMs} ms`);
};
const stages = async (fn: () => Promise<unknown>) => {
  const before = await app.json<Record<string, number>>("/tally");
  await fn();
  const after = await app.json<Record<string, number>>("/tally");
  // one console.log for the whole table: the kernel can drop trailing lines
  // when a cell ends right after a burst of separate writes
  const rows = Object.keys(after).filter((k) => k.startsWith("wf:")).map((k) => `${k.slice(3).padEnd(19)}${after[k] - (before[k] ?? 0)}`);
  console.log(["", "stage              runs", ...rows].join("\n"));
};
// read the tally baseline *before* /create: the instance starts running at once,
// and the top-of-run increment would otherwise land before the baseline is read
let first = "";
await stages(async () => {
  first = (await app.json<{ id: string }>("/create?text=the+quick+brown+fox+jumps+over+the+lazy+dog")).id;
  console.log("instance", first);
  console.log("output:", (await watch(first)).output);
});
```
`const workflowRouter = {`68 lines```
reloaded processes (4641 bytes)
instance e85682e6-8a3b-4a52-9c1c-ac77af550288
  +0.0s  running
  +0.5s  waiting
  +4.6s  running
  +5.1s  complete
output: { words: 9, longest: "quick", storedAt: 1790463403510 }
stage              runs
after sleep        1
before sleep       2
step store         1
step summarize     1
top of run()       2
```
The instance went `running → waiting → complete` and returned its summary, but the trace does not match a single pass through `run()`. The top of `run()` and the code before the sleep ran **twice**; the two steps and the code after the sleep ran once. Nothing crashed. This is the rule that Step 06 says *"decides whether your workflow is correct"*, and celld applied it in the ordinary course of a sleep:

A running workflow is stored as its steps. After a crash,

`run()`is replayedfrom the start: completed steps return their stored results instantly, and everythingoutsidea step runs again.

Map the numbers onto the article's diagram. First execution: the top of `run()` runs, `summarize` runs and stores its result, the code before the sleep runs, and `step.sleep` parks the instance. That is the `waiting` state; the instance is not resident while it waits. When the sleep is due, celld continues the instance the only way it can: by replaying `run()` from the top. The top-of-run code runs again, `summarize` returns its stored result without calling its callback (count still 1), the before-sleep code runs again, the sleep is already satisfied, and then the after-sleep code and `store summary` run for the first time. *"Everything meaningful goes inside a step, and every step must be idempotent."* The article frames replay as what happens *after a crash*. On celld it is also what happens after every durable sleep, as this run shows, so a side effect outside a step is not a rare-failure bug but an every-run bug.

The celld-specific contract from the workflows service page, as Step 06 summarizes it:

- **Limits.**A step result, an event payload, and the workflow parameters are each capped at 1 MiB; work outside a step cannot stay pending longer than 60 seconds.- *"Pass references (an R2 key, a D1 row) between steps, not payloads."*A- `workflows`entry cannot carry- `schedules`,- `limits`, or a- `script_name`naming another script, and the REST API and- `wrangler workflows`do not operate against celld.
- **Defaults.**- `step.do()`retries 5 times with a 10-second delay and exponential backoff, 10 minutes per attempt, and- `NonRetryableError`stops the loop;- `waitForEvent()`times out after 24 hours; an instance within an hour of its next alarm stays resident (- `CELLD_ALARM_RESIDENT_MS`).
- **Retention.**A successful or failed instance is kept 30 days by default; each duration in the- `retention`option can be at most 30 days; completed runs can be deleted manually.
- **Lifecycle.**The page lists no difference for- `pause()`,- `resume()`, or- `restart()`, so celld intends Cloudflare's behavior.- `restart()`works here, and it is- *not*a replay: it starts the instance over with no stored steps, so- `summarize`'s callback runs again. A replay would have returned its stored result.
- **Not available:**rollback, sensitive step results,- `ReadableStream`step results. And the article's warning stands: whether- `create()`with a terminal instance's ID replaces it or is refused,- *"verify against your installed release rather than assuming either way."*

```
await stages(async () => {
  console.log("restart →", await app.json<Status>(`/restart?id=${first}`));
  await watch(first);
});
```
`await stages(async () => {`4 lines```
restart → { status: "running", rollback: null }
  +0.0s  running
  +0.3s  waiting
  +4.4s  running
  +5.0s  complete
stage              runs
after sleep        1
before sleep       2
step store         1
step summarize     1
top of run()       2
```
## Part 4—Cron triggers (Chapter 6 "Cron, alarms, and WebAssembly")

*"Cron triggers run the  scheduled handler on celld's own durable alarms, one minute resolution in UTC, exactly once per occurrence fleet-wide. One handler runs at a time per script; a handler can run late but never early. A thrown handler is retried with backoff (starting at 4 seconds, doubling, abandoned after 6 failures, and only the expression that threw), and controller.noRetry() cancels the retry."* And the convention to remember: 

**day-of-week 1 is Sunday**, the same as Cloudflare and opposite to most cron dialects.

Declare `triggers.crons` and add a `scheduled(controller, env, ctx)` handler to the Worker. This one records each firing in `Tally` under a key carrying the scheduled minute, an RPC call from the cron handler like the one from the workflow. Then the notebook waits for the next minute boundary: up to 65 seconds, once.

```
interface ScheduledController { cron: string; scheduledTime: number; noRetry(): void }
const cronWorker = {
  fetch: workflowRouter.fetch,
  async scheduled(controller: ScheduledController, env: WorkflowEnv): Promise<void> {
    const tally = env.TALLY.get(env.TALLY.idFromName("main"));
    await tally.increment(`cron ${controller.cron} @ ${new Date(controller.scheduledTime).toISOString()}`);
    await tally.increment("cron firings");
  },
};
Object.assign(wrangler, { triggers: { crons: ["* * * * *"] } });
await app.ship({ classes: [Tally, ChatRoom, ReportBuilder], worker: cronWorker, wrangler });
const shipped = Date.now();
console.log("shipped at", new Date(shipped).toISOString(), "— next minute boundary in", 60 - new Date(shipped).getUTCSeconds(), "s");
app.clearLogs();
let fired: Record<string, number> = {};
while (Date.now() - shipped < 70_000) {
  fired = await app.json<Record<string, number>>("/tally");
  if (fired["cron firings"]) break;
  await sleep(2000);
  if ((Date.now() - shipped) % 10_000 < 2000) console.log(`  …${((Date.now() - shipped) / 1000).toFixed(0)} s, not yet`);
}
console.log(`after ${((Date.now() - shipped) / 1000).toFixed(0)} s:`, Object.fromEntries(Object.entries(fired).filter(([k]) => k.startsWith("cron"))));
console.log("node log lines mentioning cron or scheduled:", app.logs({ grep: /cron|scheduled/i }).split("\n").filter(Boolean).length);
```
`interface ScheduledController { cron: string; scheduledTime: number; noRetry(): void }`26 lines```
reloaded processes (4898 bytes)
shipped at 2026-09-26T22:56:50.102Z — next minute boundary in 10 s
  …10 s, not yet
after 10 s: { "cron * * * * * @ 2026-09-26T22:57:00.000Z": 1, "cron firings": 1 }
node log lines mentioning cron or scheduled: 2
```
The key names the minute it was scheduled for, exactly on the boundary, and it fired once. celld's node log says little or nothing about it (the count above is 2 in this run and was 0 on another machine): a cron firing is an ordinary durable alarm on an internal cell, and the `--logs` output records lifecycle, not handler invocations. If you want cron firings visible, record them yourself, as the handler here does.

## Part 5—Entities, not processes (Chapter 7)

Two primitives have now run in this notebook, and the article's framing is the test for which one a problem wants:

A cell models an

entity: a named unit with state that persists indefinitely, such as a concert, a user, or a document. A durable-execution engine (Temporal, Restate, Azure Durable Functions) models aprocess: a sequence of steps that ends, such as an order pipeline. celld's Workflows implementation is the process primitive built on the entity primitive, and its replay semantics (see Chapter 6) are exactly the tradeoff that primitive carries: steps must be idempotent, because a crash re-runs them. Pick the shape that matches the problem rather than the tool you have.

Look back at what each one did here. `ChatRoom` has no end: it accumulates a log, holds connections, wakes on demand, and is addressed by name forever. `ReportBuilder` has an end and a result: `create()` returned an ID, the instance walked through states, and `status()` finally carried its output. Its progress lived in stored steps, not in a handler's memory. That is why the durable sleep could not lose it, and also why the code between steps ran twice.

The three workloads Chapter 7 says the entity model fits (real-time applications, agents, sharded web applications) are all entities. The one that is neither obviously a process nor a chat room is the **agent**: *"Each AI agent is one cell holding memory, schedule, and inbox in its own SQLite; an idle agent hibernates to the bucket, so a large agent fleet costs almost nothing between events."* An agent is a long-lived entity that drives itself through short bursts of process, and the durable alarm is what lets it do that without a Workflow and without staying resident. That is the exercise.

### Exercise—a self-driving agent cell

Implement `Agent`: an inbox in SQL, fed by `POST /enqueue {"task": …}`, and an `alarm()` that processes **one** task per firing and then reschedules itself while the inbox is non-empty. The cell hibernates between firings; nothing lives in memory; the constructor stays trivial.

- `POST /enqueue`inserts the task into- `inbox`and,- **only if no alarm is pending**(- `getAlarm()`is- `null`), schedules one- `delayMs`ahead. Three enqueues in a row must schedule exactly one alarm.
- `alarm()`takes the oldest- `inbox`row, moves it to- `done`with a timestamp, and, if- `inbox`still has rows, calls- `setAlarm(Date.now() + delayMs)`. If the inbox is empty it sets nothing, and the cell goes quiet.
- `GET /`reports- `{ inbox, done, alarm }`(task lists and- `getAlarm()`); it is written for you.

The checker runs the class in the kernel (where `fakeCtx()` records alarms and you call `alarm()` by hand), then ships it and lets celld's alarms drive it. Alarms are durable and covered by the acknowledgement gate (Step 04; the rule itself is Chapter 4), so on a fleet this loop survives a node loss between firings.

**Hints**, from a nudge to a near-solution. Try the exercise first, then read only as far as you need.

- **Concept.**The alarm is the loop.- `/enqueue`only starts it; each- `alarm()`does one unit of work and decides whether there is a next one. Checking for a pending alarm is what keeps three enqueues from scheduling three alarms.
- **API.**- `await this.ctx.storage.getAlarm()`returns a timestamp or- `null`.- `this.ctx.storage.sql.exec(sql, …bindings)`runs a statement;- `.toArray()[0]`reads the first row, and- `.one().n`reads a- `COUNT(*) AS n`.
- **Sketch.**- `/enqueue`: insert the task, then set an alarm- `Agent.delayMs`ahead only if- `getAlarm()`is- `null`.- `alarm()`: select the oldest- `inbox`row (return if there is none), insert it into- `done`with- `Date.now()`, delete it from- `inbox`, count what is left, and set the next alarm only if that count is above zero.

Full worked solutions are intentionally not included. The checker cell is the verification: every line turns from ✗ to ✓ once the implementation is right.

```
class Agent extends DurableObject<Env> {
  static delayMs = 300;
  #schema() {
    this.ctx.storage.sql.exec("CREATE TABLE IF NOT EXISTS inbox (id INTEGER PRIMARY KEY, task TEXT NOT NULL)");
    this.ctx.storage.sql.exec("CREATE TABLE IF NOT EXISTS done (id INTEGER PRIMARY KEY, task TEXT NOT NULL, at INTEGER NOT NULL)");
  }
  async fetch(request: Request): Promise<Response> {
    this.#schema();
    if (request.method === "POST" && new URL(request.url).pathname === "/enqueue") {
      const { task } = await request.json() as { task: string };
      // TODO: insert `task` into inbox; if getAlarm() is null, setAlarm(Date.now() + Agent.delayMs)
      return new Response("not implemented", { status: 501 });
    }
    return Response.json({
      inbox: this.ctx.storage.sql.exec<{ task: string }>("SELECT task FROM inbox ORDER BY id").toArray().map((r) => r.task),
      done: this.ctx.storage.sql.exec<{ task: string }>("SELECT task FROM done ORDER BY id").toArray().map((r) => r.task),
      alarm: await this.ctx.storage.getAlarm(),
    });
  }
  async alarm(): Promise<void> {
    this.#schema();
    // TODO: move the oldest inbox row to done (with at = Date.now());
    //       if inbox is still non-empty, setAlarm(Date.now() + Agent.delayMs)
  }
}
```
`class Agent extends DurableObject<Env> {`28 lines```
// ── checker ────────────────────────────────────────────────────────────────
type AgentState = { inbox: string[]; done: string[]; alarm: number | null };
const check = (label: string, ok: boolean, detail = "") => console.log(`${ok ? "✓" : "✗"} ${label}${detail && !ok ? ` — ${detail}` : ""}`);
const enqueueReq = (task: string) => new Request("http://cell/enqueue", { method: "POST", body: JSON.stringify({ task }) });
const agentCtx = fakeCtx("agent-1");
const agent = new Agent(agentCtx, {} as Env);
for (const t of ["read inbox", "draft reply", "file report"]) await agent.fetch(enqueueReq(t));
let st = await (await agent.fetch(new Request("http://cell/"))).json() as AgentState;
check("kernel: three tasks queued", st.inbox.length === 3, JSON.stringify(st));
check("kernel: exactly one alarm scheduled for three enqueues", agentCtx.alarms.length === 1, `alarms recorded: ${agentCtx.alarms.length}`);
for (let i = 1; i <= 3; i++) {
  await agentCtx.storage.deleteAlarm();      // celld clears the alarm before calling alarm(); mimic that
  await agent.alarm();
  st = await (await agent.fetch(new Request("http://cell/"))).json() as AgentState;
  check(`kernel: firing ${i} processed one task`, st.done.length === i && st.inbox.length === 3 - i, JSON.stringify(st));
}
check("kernel: rescheduled while non-empty, then stopped", agentCtx.alarms.length === 3 && st.alarm === null, `alarms recorded: ${agentCtx.alarms.length}, pending: ${st.alarm}`);
check("kernel: FIFO order", st.done.join(",") === "read inbox,draft reply,file report", st.done.join(","));
wrangler.durable_objects.bindings.push({ name: "AGENT", class_name: "Agent" });
wrangler.migrations.push({ tag: "v3", new_sqlite_classes: ["Agent"] });
await app.ship({ classes: [Tally, ChatRoom, ReportBuilder, Agent], worker: cronWorker, wrangler });
const agentUrl = "/do/AGENT/agent-1";
for (const t of ["read inbox", "draft reply", "file report"]) await app.fetch(`${agentUrl}/enqueue`, { method: "POST", body: JSON.stringify({ task: t }) });
const t0 = Date.now();
let live = await app.json<AgentState>(`${agentUrl}/`);
check("celld: queued with one alarm pending", live.inbox.length === 3 && live.alarm !== null, JSON.stringify(live));
while (Date.now() - t0 < 5000 && live.inbox.length > 0) { await sleep(200); live = await app.json<AgentState>(`${agentUrl}/`); }
check(`celld: alarms drained the inbox in ${((Date.now() - t0) / 1000).toFixed(1)} s`, live.inbox.length === 0 && live.done.length === 3 && live.alarm === null, JSON.stringify(live));
```
`type AgentState = { inbox: string[]; done: string[]; alarm: number | null };`31 lines```
✗ kernel: three tasks queued — {"inbox":[],"done":[],"alarm":null}
✗ kernel: exactly one alarm scheduled for three enqueues — alarms recorded: 0
✗ kernel: firing 1 processed one task — {"inbox":[],"done":[],"alarm":null}
✗ kernel: firing 2 processed one task — {"inbox":[],"done":[],"alarm":null}
✗ kernel: firing 3 processed one task — {"inbox":[],"done":[],"alarm":null}
✗ kernel: rescheduled while non-empty, then stopped — alarms recorded: 0, pending: null
✗ kernel: FIFO order
reloaded processes (6071 bytes)
✗ celld: queued with one alarm pending — {"inbox":[],"done":[],"alarm":null}
✗ celld: alarms drained the inbox in 0.0 s — {"inbox":[],"done":[],"alarm":null}
```
## Clean up

Stop the node. State stays in `.celld/dev` under `app.dir`; the next `app.start()` without `clean: true` resumes it, including the workflow instances, which are inside their 30-day retention.

`await stopAll();``await stopAll();`1 linecelld dev stopped (exit 0)

## Where next

This was the process side of the cell model: RPC as the natural call shape between a Worker and its cells (Chapter 6); hibernation watched from the log, with the constructor running again on every wake and hibernatable sockets surviving it (Chapter 2, Step 04); Workflows and the replay rule made visible through a durable sleep (Step 06); cron as a durable alarm with minute resolution and Sunday as day 1; and the entity/process distinction that puts an agent on the cell side of the line (Chapter 7).

What the three notebooks deliberately leave to the articles is everything about a **fleet**: the bucket as coordinator, ownership and fencing, replication as LTX segments, balancing of hibernated cells, and failover (Chapter 3–05 and Steps 08–10 of the guide). Those are operational, and `celld dev`'s local store is a single node. But the lifecycle events in this notebook's log output, and their siblings (`eviction_durability_barrier`, `restore_plan`, `cell_isolate_startup_timing`), are the same ones a fleet node writes when a cell moves. Read those chapters with this notebook's log output beside you.

**Maintenance.** This export is frozen at celld v0.6.0, executed 2026·09·26. Re-execute it on each new celld release and note what changed: upstream moves fast, with five releases (v0.4.0 through v0.6.0) between 2026·08·28 and 2026·09·26.

This lab draws on: celld: documentation at v0.6.0 (3 entries). The full entries are in the Bibliography.


---

<!-- https://tlockney.github.io/celld-book/review.html -->

## How to use this sheet

Part A is for looking things up: the rules and numbers you reach for while building or operating, compressed from Part II and Chapter 9. Part B is for testing yourself after reading the book. Each question targets a mistake the book warns about; answer it in your head, then check the answer in § 04. Each answer names the place in the book that explains it.

## Part A · Quick reference

### The compatibility line

**If Cloudflare builds a function on Durable Objects, celld can carry it.** A configuration or binding that is not available must fail loudly, at deploy or first use; a silent gap is a bug.

| Carried (graded Yes) | Not carried | 
|---|---|
| Workers, Durable Objects, facets, static assets, cron, Dynamic Workers, KV, Queues, D1, Workflows, R2 | Workers AI, Vectorize, Hyperdrive, Browser Rendering, Email, Python Workers, BroadcastChannel | 
| Containers and the Sandbox SDK: Experimental | Cache API: Partial, an always-miss cache | 

Part II's introduction and Chapter 6 have the full surface.

### Durability: which proof acknowledges a write

A write is acknowledged only once it survives a failure (**RPO=0**). What proves it depends on the fleet:

| Fleet | Proof | What the write waits for | 
|---|---|---|
| One node | Bucket proof | One storage round trip to the bucket | 
| Two or more nodes | Fleet proof | Every follower (one or two, never the owner) has fsynced it; the bucket upload finishes afterwards | 

`CELLD_DURABILITY` selects the mode: `fleet` (the default) or `bucket`. A single node that asks for `fleet` falls back to bucket proof. A fleet of three or more nodes holds three copies of an acknowledged write. Chapter 3.

### One writer, everywhere

| Unit | Rule | 
|---|---|
| Cell | One owner node at a time, fenced by the epoch in its ownership record | 
| D1 database | One writer; more capacity means more databases, never a bigger one | 
| KV namespace | One writer; add namespaces, not writers | 
| Queue | One writer, and one consumer script | 

Shard by adding entities, never by growing one. Chapter 7.

### Environment variables

Copied from Chapter 5.

| Variable | Purpose | 
|---|---|
| `CELLD_BUCKET` | Fleet bucket (+ optional key prefix). Same as `--bucket`. | 
| `S3_ENDPOINT`,`AWS_REGION`,`AWS_*` | S3-compatible endpoint and credentials (standard AWS chain; EKS Pod Identity supported). | 
| `GOOGLE_*` | Google credentials for a `gs://`bucket (ADC or service-account key). | 
| `AZURE_*` | Azure account/identity for an `az://`bucket (exactly one credential family). | 
| `CELLD_DURABILITY` | Durability mode: `fleet`(default) or`bucket`. Fleet proof needs ≥ 2 nodes. | 
| `CELLD_ADDR`/`CELLD_INTERNAL_ADDR`/`CELLD_ADVERTISE` | Public listener, internal peer/operator listener, advertised address. | 
| `CELLD_ACTIVATIONS` | Concurrent cold-cell activations (default 8 per CPU, at least 16 and at most 128; a cold activation mostly waits on the store, so the default sits above the CPU count). | 
| `CELLD_MAX_RESIDENT_CELLS` | Hard resident-cell cap, enforced at admission. | 
| `CELLD_IDLE_EVICT_S` | Seconds without work after which an idle resident cell hibernates (unset: only pressure or the cap removes it). Balancing moves hibernated cells only. | 
| `CELLD_PLACEMENT_WEIGHT`/`CELLD_REBALANCE_INTERVAL_MS` | This node's ownership share relative to its peers (default: CPU count) and the fleet-sample interval (default 5,000; 0 disables balancing). | 
| `CELLD_MAX_CELL_REQUESTS` | Concurrent fetch limit for one Durable Object (default 64). A Queue broker has its own fixed limits: 256 concurrent producer calls, 64 per transaction, four overlapping proofs. | 
| `CELLD_MAX_REQUEST_BODY_BYTES` | Body limit for a public Worker request or direct DO request (default 1 GiB). | 
| `CELLD_MAX_RSS_MB` | Memory threshold for pressure shedding (default 80% of available memory; accounts for cgroup memory on Linux). | 
| `CELLD_TTL_MS` | Node-lease lifetime (default 10,000 ms). | 
| `CELLD_OPERATION_DEADLINE_MS` | Deadline for a non-restore operation (default 15,000). | 
| `CELLD_DEPLOY_POLL_S`/`CELLD_DEPLOY_MAX_AGE_S` | Deployment-adoption poll interval (30s) and forced-move age (60s). | 
| `CELLD_SHUTDOWN_TOTAL_MS`/`CELLD_RELEASES` | Total stop bound (40s; the drain-token wait and no-progress bound derive from it) and concurrent handoffs (default 128). | 
| `CELLD_LTX_COMPACTION` | 1 (default) creates additive L1 objects so a takeover reads tens of objects instead of thousands. | 
| `CELLD_LTX_PAGED`/`CELLD_LTX_PAGED_MIN_MB`/`CELLD_LTX_HYDRATE_MBPS` | Paged restore (default on) for chains above the threshold (default 256 MiB), and the background fill rate for the paged file (default 16; 0 keeps it sparse). | 
| `CELLD_RECOVERY_RETRY_MS`/`CELLD_RECOVERY_RETRIES` | Node-log recovery retry pacing (defaults 1,000 and 240); recovery reads bundles in 512 MiB windows and checkpoints every 32 cell epochs. | 
| `CELLD_WAKER_TICK_MS` | Interval of the fleet waker's alarm cleanup pass (default 60,000). | 
| `CELLD_ALARM_RESIDENT_MS` | How close to its next alarm a cell (a Workflow instance, say) stays resident rather than hibernating (default one hour). | 
| `CELLD_DOCKER`/`CELLD_CONTAINER_PLATFORM`/`CELLD_CONTAINER_RUNTIME` | Containers: the container CLI used to build and pull (default `docker`; Podman works), the deploy build platform (default`linux/amd64`), and a node-wide OCI runtime such as`runsc`or`kata`. | 
| `CELLD_OTEL` | `0`off,`1`Parquet to the bucket, or an OTLP/HTTP collector base URL. | 
| Rejected at startup | `CELLD_OUTPUT_GATE`(the gate is always on; celld always waits for the configured durability proof),`CELLD_STORAGE_PROBE`,`CELLD_SHUTDOWN_DRAIN_MS`,`CELLD_DRAIN_TOKEN_WAIT_MS`,`CELLD_WORKER_LOADER`,`CELLD_MAX_LOADED_WORKERS`,`CELLD_OTEL_SINK`,`CELLD_AI_BINDING`,`CELLD_AI_URL`,`CELLD_REBALANCE_BATCH_CELLS`, and a handful of older tuning knobs. A node with any of them set does not start. | 

### Upgrade cliffs

Seven documented transitions. A full stop means stop every old node before starting any new one.

| Transition | Rule | 
|---|---|
| v0.1.0 → v0.2.0 | Full stop | 
| v0.2.1 → v0.3.0 | Rolling; never start a v0.2.x binary after v0.3.0 has run unless the shutdown log shows `node-log close: sealed epoch` | 
| v0.3.0 → v0.4.0 | Full stop | 
| v0.4.0 → v0.4.1 | Rolling; never start a v0.4.0 binary once the fleet begins paging | 
| v0.4.1 → v0.5.0 | Full stop, with a written procedure; the only rollback is the stopped-fleet backup | 
| v0.5.0 → v0.5.1 | Rolling | 
| v0.5.1 → v0.6.0 | Full stop under `fleet`durability; rolling under`bucket` | 

**Never roll back by starting an old binary against an upgraded bucket.** Chapter 5 has each transition's reason and the v0.5.0 procedure.

### Limits worth memorizing

| Area | Limit | 
|---|---|
| Isolate heap | 128 MB V8 heap by default ( `CELLD_V8_HEAP_LIMIT_MB`);`acceptWebSocket()`throws past 90% | 
| WebSocket input | 1 MiB per-isolate input budget per non-terminal frame | 
| Workflows | Step result, event payload, and workflow parameters each at most 1 MiB; non-step work pending at most 60 s; instances kept 30 days by default | 
| `step.do()`defaults | 5 retries, 10-second delay, exponential backoff, 10 minutes per attempt | 
| D1 | Results capped at 100,000 rows / 32 MiB per binding result | 
| Queue messages | At most 128,000 bytes each; `sendBatch()`at most 100 messages and 256,000 bytes;`delaySeconds`up to 86,400 | 
| Queue retention | Four days, not configurable | 
| Queue consumer | `max_batch_size`10 (max 100),`max_batch_timeout`5 s (max 60),`max_retries`3,`max_concurrency`up to 250 | 
| KV | Key at most 512 bytes; value at most 25 MiB; metadata at most 1,024 bytes; a value above 1 MiB needs the fleet bucket | 
| R2 | A conditional write cannot stream a body above 8 MiB; `delete(keys)`takes up to 1,000 keys | 
| Dynamic Workers | At most 256 live per process, 255 per script generation; module sources at most 64 MiB | 

The KV key, value, and metadata limits come from the celld.dev KV service page; the rest are in Chapter 6 and Chapter 9.

### Fail loudly

celld's rule for everything it does not model is to refuse it where you will see it. For deploys, that means:

- **An unknown Wrangler key stops the deploy**with an error naming the key (- `routes`, for example).
- **A binding celld does not carry fails**at deploy or on first use, never silently.
- **Queue consumer settings are validated at deploy time**, so a bad value fails the deployment rather than the first delivery.
- **Two scripts consuming one queue**is a deployment that fails.
- **A node with a rejected environment variable set does not start**, and a store that fails the startup probe's contract checks stops the node.

## Part B · Self-quiz

Answer each in a sentence or two, then check § 04.

### The model

- What is a cell, in Cloudflare's terms, and what does it own?
- A handler reads a counter, awaits an outbound `fetch()`, then writes the counter back. Why does this lose updates under concurrent requests, and what are the two fixes?
- Must you `await`a storage write before responding, to be sure the client never sees an unpersisted success?
- What exactly survives hibernation, and what does not?
- Is idle eviction on by default, and why does the answer matter for balancing?
- Which primitive models an entity, and which models a process?

### Durability and fencing

- What proves a write durable on a one-node fleet, and on a fleet of two or more nodes?
- What does `CELLD_DURABILITY=fleet`do on a single node?
- How does a node acquire a cell, and what does every activation do to the epoch?
- Why does the ownership check read the ownership record instead of comparing clocks?
- What does a node do when it can no longer renew its node lease?

### The bucket

- Which stores are fleet-qualified, and what does the startup storage probe actually test?
- Can you turn the storage probe off?
- What does the bucket hold, and why does that make its credentials special?

### Operating a fleet

- Which cells does ownership balancing move, and how is a node's share decided?
- How do you upgrade a fleet from v0.5.1 to v0.6.0?
- After upgrading to v0.5.0, how do you roll back?

### Services

- Does KV on celld have an edge cache?
- A queue message fails more than `max_retries`times and the consumer names no dead-letter queue. What happens, and how long does celld wait between attempts?
- What happens to a deployment in which a declared queue consumer's script has no `queue()`handler?
- Two R2 objects hold identical bytes. Do they have the same `version`on the fleet bucket, and on`celld dev`'s local store?
- How do you give a browser a public link to an object in an R2 binding?
- Why must a facet write never be assumed to commit with the root object's transaction?
- A workflow sleeps durably, then continues. Is that resumption the same mechanism as recovery after a crash?
- In a cron expression on celld, what day is day-of-week 1?
- You ship a D1 migration file with `celld dev`. Is the table there afterwards?

## Answers

Each answer is collapsed. Open one only after answering it.

## 1. What a cell is

A cell is a Durable Object instance: a small server with a name and a private SQLite database. It serves HTTP, holds WebSocket connections, sets alarms, and makes outbound connections. Chapter 2.

## 2. The lost update

One thread per cell means two requests never run at the same instant, but a second request can interleave while the first *awaits* anything that is not a storage call. Both requests read the same old value and both write value + 1. Fix it by keeping the read-modify-write free of non-storage awaits (storage calls are synchronous and never interleave), or by wrapping the work in `blockConcurrencyWhile()`. Lab 1, Part 2 shows ten racy increments all returning 1, and the gated version returning 1 through 10. Chapter 2.

## 3. Awaiting writes

No. The **output gate** holds each write response, and every outbound effect, until a durability proof covers the write. A client cannot see a success for a write that was not made durable. Chapter 4; Chapter 1 § 03.

## 4. What survives hibernation

Two things separate a hibernated cell from a cold one: its hibernatable WebSocket clients stay connected, and it stays on its node. Memory does not survive. The constructor runs again on every wake, including each message to a hibernated room, so restore state from storage inside the handler and keep the constructor trivial. Chapter 2; Lab 3, Part 2.

## 5. Idle eviction

No, it is opt-in: `CELLD_IDLE_EVICT_S` sets the idle seconds, and without it only memory pressure or the residency cap removes a resident cell. It matters because balancing moves only hibernated cells, so a fleet without idle eviction balances only the cells that hibernate on their own. Chapters 2 and 5.

## 6. Entity or process

A cell (a Durable Object) models an entity: a named unit whose state persists indefinitely. A workflow models a process: steps that start, run, and finish. celld's Workflows are the process primitive built on the entity primitive. Chapter 1 § 06; Chapter 7.

## 7. Proofs by fleet size

One node: a **bucket proof**, one storage round trip to the bucket. Two or more nodes: a **fleet proof**, which answers once every follower has fsynced the write; the bucket upload finishes afterwards. Chapter 3.

## 8. Fleet durability on one node

It requests the fleet posture and does not get it: the node falls back to bucket proof, because a fleet proof needs at least two nodes. Chapter 3.

## 9. Acquiring a cell

With a conditional write to the cell's ownership record in the bucket: create when no record exists, compare-and-swap when one does. The bucket accepts only one such write. Every activation advances the epoch, a takeover and a local wake alike, and the cell's data is written under that epoch's prefix. Chapter 4.

## 10. Records, not clocks

After a bucket proof, celld re-reads the ownership record and acknowledges only if it still names this node at this epoch. Reading the record means a paused process or a skewed clock cannot pass the check; a clock comparison could be fooled by either. Chapter 4.

## 11. Losing the lease

It fences itself. When its published lease expiry passes, it stops each active cell, fails incomplete requests, logs a line starting `SELF-FENCE:`, and exits with code 3, so it cannot keep serving state it can no longer prove. Chapter 4.

## 12. Qualified stores and the probe

Fleet-qualified: Amazon S3, Cloudflare R2, Google Cloud Storage, Tigris, and Azure Blob Storage. MinIO passes the test but is not qualified; Backblaze B2, Hetzner, and DigitalOcean Spaces are not. The startup probe runs conditional writes that must succeed and fail in the right places, plus a ranged read that must return exactly the requested bytes. A clear contract violation stops the node; an ambiguous failure gets three attempts and then a warning. Chapter 3.

## 13. Disabling the probe

No. The startup probe cannot be disabled: `CELLD_STORAGE_PROBE` is on the rejected list, and a node with it set does not start. `celld diagnose --read-only` skips the probe only for that on-demand check. Chapters 3 and 5.

## 14. What the bucket holds

Deployments (with container images), cell state, ownership records, node leases, the shared peer-authentication secret, the fleet capacity sample, alarm wake entries, large KV values, and all R2 objects. Whoever holds the bucket credentials controls the fleet. Chapters 3 and 8.

## 15. Balancing

Only hibernated cells move. A node's target is the fleet's owned cells divided by weight, where `CELLD_PLACEMENT_WEIGHT` defaults to the CPU count. `POST /rebalance/pause` and `/resume` on any internal listener govern the whole fleet. Chapter 5.

## 16. v0.5.1 to v0.6.0

Under `fleet` durability, as a full stop: stop every v0.5.1 node, then start the v0.6.0 nodes, because a v0.6.0 node needs a follower that speaks the ranged `CLT2` log-tail format and refuses to start in a mixed fleet. A fleet on `CELLD_DURABILITY=bucket` has no followers and can roll. Chapter 5.

## 17. Rolling back from v0.5.0

Never by starting an old binary against the upgraded bucket. The only rollback is the backup taken while the fleet was stopped, and it loses every write made after it. Chapter 5.

## 18. KV edge cache

No. `cacheTtl` has no effect and `cacheStatus` is `null`: KV on celld is a durable store with KV's API, not a CDN. Reads route to the namespace's cell. Chapter 6; Lab 2, Part 2.

## 19. Past max_retries without a dead-letter queue

celld deletes the message. Between attempts, a retried message becomes visible again after the `delaySeconds` passed to `retry()` or `retryAll()`, or the consumer's `retry_delay` by default; celld adds no exponential backoff, so an application that wants one computes it from `message.attempts`. Chapter 6; Chapter 9, Step 05.

## 20. A consumer without a handler

The script fails to load and the node goes down: Lab 2, Part 5 shows the local node exiting with `queue consumer has no queue handler`. Declare a consumer only in a script that exports `queue()`. Chapter 6.

## 21. R2 versions

On a store that reports no version of its own, the ETag becomes the version, so identical content produces the same version. `celld dev`'s local store instead numbers each write from a store-wide counter, so identical bytes get different versions there (Lab 2, Part 4 showed 74 and 75). Either way, never use a version to count writes, and use `checksums.md5` as the content hash. Chapter 6; Chapter 9, Step 07.

## 22. Publishing an R2 object

Through a Worker. celld serves a bucket through the binding only: there is no public bucket URL, no presigned URL, and no S3 endpoint into an R2 binding. Chapter 6; Chapter 9, Step 07.

## 23. Facet writes

Each facet lives in its own SQLite file with its own replication stream, so a facet write commits in the facet's database. Rolling back a root transaction does not undo a facet call made inside it. State that needs one atomic commit belongs in one database. This changed in v0.6.0. Chapter 6; Appendix C.

## 24. Sleep versus crash

The same mechanism. `run()` executes again from the top, completed steps return their stored results, and everything outside a step runs again. After a durable sleep nothing failed, but the re-execution is identical: Lab 3, Part 3 counted the top of `run()` running twice across one sleep while each step ran once. Chapter 6; Chapter 9, Step 06.

## 25. Cron day-of-week

Sunday. Day-of-week 1 is Sunday, as on Cloudflare and opposite to most cron dialects. Cron runs in UTC at one-minute resolution, exactly once per occurrence fleet-wide. Chapter 6.

## 26. D1 migrations on celld dev

No. A `celld dev` deploy does not apply migrations, and `celld d1 migrations apply` needs a fleet. Locally, run the SQL from the Worker (for example `CREATE TABLE IF NOT EXISTS`), as celld's own `examples/d1` does. Chapter 9, Step 07; Lab 2, Part 3.

This chapter draws on: celld: documentation at v0.6.0 (1 entry). The full entries are in the Bibliography.


---

<!-- https://tlockney.github.io/celld-book/glossary.html -->

The terms this book uses, each defined once and used the same way in all six parts. The last column names the section that explains it.

Tbl. 1

| Term | Meaning | See | 
|---|---|---|
| Actor | A unit of computation with private state that interacts only by asynchronous messages. In response to a message it can send messages, create actors, and choose its behavior for the next message. | Chapter 1 § 04 | 
| Binding | A permission and an API in one object on `env`, through which a Worker reaches a platform resource with no credential in its code. | Chapter 1 § 02 | 
| Bucket proof | The durability proof with one node: the write waits for one storage round trip to the bucket before it is acknowledged. | Chapter 3 | 
| Cell | A Durable Object instance: a small server with a name and a private SQLite database. One per user, document, room, or agent. | Chapter 2 | 
| Durability mode | `CELLD_DURABILITY`:`fleet`(the default) acknowledges on a fleet proof,`bucket`on a bucket proof. A single node falls back to bucket proof. | Chapter 3 | 
| Durable execution | Code that survives crashes: each step's result is recorded in a persistent log, and after a failure the code is replayed with recorded steps answered from the log. | Chapter 1 § 05 | 
| Ensemble | A cell's owner together with the followers it sends each write to. | Chapter 3 | 
| Entity | A named unit whose state persists indefinitely, such as a user, room, or document. A cell is an entity; contrast process. | Chapter 1 § 06 | 
| Epoch | The fencing number in an ownership record. Every activation advances it, and replicated data is written under an epoch prefix, `cells/<cell>/ltx/e<epoch>/`. | Chapter 4 | 
| Facet | A child object attached to a Durable Object through `ctx.facets`, with its own SQLite database and replication stream, sharing the root's ownership record and epoch. | Chapter 6 | 
| Fleet | The set of nodes sharing one fleet bucket. | Chapter 3 | 
| Fleet bucket | The object-storage bucket a fleet shares. It holds deployments, cell state, ownership records, node leases, and R2 objects, and it is the root of authority. | Chapter 3 | 
| Fleet proof | The durability proof with two or more nodes: the write is acknowledged once every follower has it on disk, and the bucket upload finishes afterwards. | Chapter 3 | 
| Follower | A node, never the owner itself, that receives a cell's writes for a fleet proof. The owner picks one or two. | Chapter 3 | 
| Hibernated | A cell evicted from memory whose hibernatable WebSocket clients stay connected and which stays on its node. Balancing moves only hibernated cells. | Chapter 2 | 
| Inactive | A cell no node holds: only an object in the bucket. Every cell starts inactive. | Chapter 2 | 
| Input gate | Cloudflare's rule that no new event reaches an object while one of its storage operations is in flight. celld gets the same effect from synchronous storage calls. | Chapter 1 § 03 | 
| Internal listener | The node listener for the peer protocol and the operator API ( `--internal-listen`). It must stay on a private network or encrypted overlay. | Chapter 8 | 
| Isolate | A V8 sandbox with its own memory and variables. One runtime process hosts many isolates, which is what makes Workers start fast. | Chapter 1 § 02 | 
| Local store | The on-disk store `celld dev`uses, in`.celld/dev`under the project. Fleet nodes cannot select it; fleets require a qualified cloud bucket. | Chapter 5 | 
| LTX segment | Litestream's replica format, in which celld ships each cell's SQLite state to the bucket. | Chapter 3 | 
| Node | One celld process. | Chapter 3 | 
| Node lease | A node's record in the bucket with an expiry ( `CELLD_TTL_MS`), renewed after one third of its lifetime. A node that cannot renew it fences itself. | Chapter 4 | 
| Output gate | The rule that holds each write response until a durability proof covers it, applied in one order across responses, outbound calls, Queue deliveries, and WebSocket sends. | Chapter 4 | 
| Owner | The node session an ownership record names: the only one that may run the cell. | Chapter 4 | 
| Ownership balancing | How the fleet evens out ownership without a coordinator: the node with the most owned cells per unit of weight hands hibernated cells to the peer furthest below its share. | Chapter 5 | 
| Ownership record | The one record per cell in the bucket that names its owner and epoch, acquired with a conditional write. | Chapter 4 | 
| Paged restore | Restoring a large cell page by page through a fault-in SQLite VFS that reads each page from the bucket on first use. | Chapter 3 | 
| Peer tunnel | The versioned plain-HTTP tunnel that carries every proxied cell call (fetch, RPC, WebSocket) from the node that took the request to the owner. | Chapter 4 | 
| Process | A sequence of steps that starts, runs, and finishes, such as an order pipeline. A workflow is a process; contrast entity. | Chapter 1 § 06 | 
| Replay | Re-running durable code from the start, answering each completed step from the log. It requires deterministic control flow and idempotent steps. | Chapter 1 § 05 | 
| Resident | A cell in memory: activewhile it does work,idlewhile it waits. | Chapter 2 | 
| RPO=0 | No acknowledged write is lost: celld does not answer a write until the data survives a failure. | Chapter 3 | 
| Self-fence | A node whose lease expiry passes stops its cells, fails incomplete requests, logs `SELF-FENCE:`, and exits with code 3. | Chapter 4 | 
| Virtual actor | An actor that always exists logically, is activated on demand and deactivated when idle, and is addressed by identity rather than location (an Orleans grain). A Durable Object is one. | Chapter 1 § 04 | 

Sources

This chapter draws on: celld: documentation at v0.6.0 (9 entries) · celld: release notes (5 entries) · Cloudflare documentation (4 entries). The full entries are in the Bibliography.


---

<!-- https://tlockney.github.io/celld-book/release-notes.html -->

§01

## Upgrade cliffs

Warning

§02

## Release notes

The chapters of this book describe v0.6.0. This table records what each release since v0.4.0 changed, so the history is in one place. The "Upgrade" column repeats the transition rule from the callout above; a change marked **behaviour** altered what a running application observes and can bite on upgrade.

Tbl. 1

| Release | Date | Upgrade from previous | Notable changes | 
|---|---|---|---|
| v0.4.0 | 2026·08·28 | Full stop from v0.3.0 | Workers KV, Queues, Workflows, and R2 arrive, graded Partialat the time. Every proxied cell call (fetch, RPC, WebSocket) moves onto one versioned plain-HTTP peer tunnel, and deployments adopt in place without a restart. The output gate applies one ordering rule across responses, outbound calls, Queue deliveries, and WebSocket sends.`celld dev`opens a local store, replacing the earlier "no local filesystem mode" guidance. Large KV values move under an epoch-qualified reference.Behaviour:the health path moves from`/__celld/health`to`/.well-known/celld/health`; update load balancers and readiness probes. Gaps at this release:`wrapKey`/`unwrapKey`, RSA signing, and HKDF/PBKDF2 derivation were unavailable; Workflow retention and manual deletion were unavailable; the store was allowed to ignore`Range`; a node whose ready gate expired reported healthy anyway; the docs said a queue consumer could not also export`fetch()`, and that`create()`with a terminal Workflow instance's ID replaced it. | 
| v0.4.1 | 2026·09·05 | Rolling; never restart v0.4.0 once paging began | Durable Object facets, HTMLRewriter, TCP sockets, EventSource, MessageChannel, an always-miss Cache, and the missing Web Crypto pieces. Paged restore through a fault-in SQLite VFS, which makes exact ranged reads a correctness requirement. Ownership balancing replaces the earlier rule that a joining node took only unowned cells, with `POST /rebalance/pause`/`/resume`. The self-fence names its cause with three distinct events, and a failed lease renewal retries before the authority expires. Nodes that restart together recover acknowledged writes, with checkpoints and a heartbeat. The Queue producer path is rebuilt: shared transactions and durability rounds, time-ordered message IDs, 7,357 sends/s on one queue (up from a few hundred).`celld diagnose`shows each node's load sample. Outbound TCP makes egress control the fleet network's job. | 
| v0.5.0 | 2026·09·15 | Full stop with a procedure (alarm wake format 2); no binary rollback | Containers and the Sandbox SDK ( Experimental). Dynamic Workers, the renamed Worker Loader, leaves the experimental tier and is declared in`wrangler.jsonc`(`worker_loaders`) rather than`CELLD_WORKER_LOADER`. Exact ranged reads become the fourth documented store requirement, and the startup probe can no longer be disabled. The compatibility page is reframed to list only differences and to gradeYes/Partial/Experimental/No. Idle eviction becomes opt-in via`CELLD_IDLE_EVICT_S`. Deploy records SHA-256 module digests;`no_bundle`projects get Wrangler's`**/*.wasm`discovery; container images live under`deploy/images/`.`celld dev`gains`--clean`,`--watch-ignore`, and`.dev.vars`. A restarting node waits behind a peer already recovering its log. An`AbortSignal`passes through a same-node RPC call. Shutdown tunables collapse into`CELLD_SHUTDOWN_TOTAL_MS`; the`CELLD_RELEASES`default rises from 8 to 128;`CELLD_ACTIVATIONS`defaults to 8 per CPU. An expired ready gate keeps readiness closed.`/state`adds`handed_off`,`rebalanced`,`rebalance_failed`,`remote_route_refreshes`, and`node_load`.`CELLD_OTEL`picks the telemetry sink;`OTEL_EXPORTER_OTLP_ENDPOINT`is no longer read. Workflow retention becomes configurable and completed runs deletable; replay is no longer listed as a celld difference. D1 migration extensions become case-insensitive; KV prefix listings get faster; internal host functions stop being exposed to application code.Behaviour:the variables listed underRejected at startupin the environment table stop a node that sets them. | 
| v0.5.1 | 2026·09·19 | Rolling | `WorkerCode.limits`enforces`cpuMs`and`subRequests`, and`WorkerCode.tails`delivers Tail Worker reports (both rejected in v0.5.0). The`celld r2`CLI (`get`,`head`,`put`,`delete`,`list`) replaces`wrangler r2 object`. The documentation splits into per-service pages.`/state`adds`allocator`,`libc_malloc`, and`deployment.isolates`;`/evict/<cell>`waits for the eviction and reports the true outcome instead of reporting success early. Wrangler`define`and`rules`are accepted. Workflow defaults are documented for the first time, along with the REST API /`wrangler workflows`exclusion. The facets page states the alarm, depth, and name-length limits.`examples/queues`exports both`fetch`and`queue`from one script.Behaviour:on Azure the R2 metadata name becomes`celld_r2`(#209). | 
| v0.6.0 | 2026·09·26 | Full stop under `fleet`durability (`CLT2`log tail); rolling under`bucket` | First beta. Each facet gets its own SQLite file and replication stream, and a facet class can come from`ctx.exports`; facets written by v0.5.1 migrate on first open. Ed25519 (also`NODE-ED25519`) and X25519 in Web Crypto.`ctx.exports`holds`default`and each entrypoint.`Headers`accepts values above U+00FF and decodes response values as UTF-8. The WebSocket output gate holds each frame only for its own proof, and a close is`wasClean: true`whenever the peer sent a close frame. R2 keeps empty key segments and percent-encodes special characters.Behaviour:a facet write no longer commits atomically with a root transaction; a D1`TEXT`value that is not valid UTF-8 decodes to U+FFFD instead of failing;`WorkerCode`requires`compatibilityDate`, a wasm module entry must be`{ wasm: bytes }`, and a relative import in a module subdirectory resolves from the importing module's name; every export of the main module must be a handler object or a class; an R2 key v0.5.1 wrote as`photos/`stays at`photos`. | 

Sources

This chapter draws on: celld: documentation at v0.6.0 (9 entries) · celld: release notes (5 entries) · Cloudflare documentation (4 entries). The full entries are in the Bibliography.


---

<!-- https://tlockney.github.io/celld-book/wal-ltx.html -->

## Why this appendix

The chapters say that celld "continuously ships each cell's SQLite state to the bucket as LTX segments" (Chapter 3), that a large cell restores "page by page through a fault-in SQLite VFS", and that a v0.6.0 node needs the ranged `CLT2` log-tail format (Appendix C). Each of those sentences leans on three older ideas: SQLite's write-ahead log, the Litestream replicator, and the LTX file format. This appendix explains them in that order, then shows what celld's replication library (`crates/ltx` in `denoland/celld`) does with them, with the details as they stand at celld v0.6.0.

Where celld departs from Litestream, the appendix says so. The departures are the interesting part: celld keeps Litestream's capture and file format but replaces its coordination with the epoch fencing, output gate, and node log that the chapters describe.

## SQLite's write-ahead log

SQLite has two ways to make a transaction atomic. In the older **rollback journal** mode, it copies each page it is about to change into a `-journal` file, then overwrites the page in place; a crash is repaired by copying the originals back. In **WAL mode** the database file is left alone during a write. Changed pages are appended to a separate `-wal` file instead, and readers combine the two. Writers no longer block readers, readers no longer block the writer, and the disk sees mostly sequential writes and fewer `fsync()` calls.

### Frames and commits

A WAL file is a 32-byte header followed by **frames**. The header carries a magic number (which also fixes the checksum byte order), the format version, the page size, a checkpoint sequence number, two random **salts**, and a checksum. Each frame is a 24-byte frame header and one page of data. The frame header records:

| Field | Meaning | 
|---|---|
| Page number | Which database page the frame holds. | 
| Commit size | The database size in pages afterthe transaction, on the frame that commits it; 0 on every other frame. | 
| Salt-1, salt-2 | Copies of the header's salts. | 
| Checksum-1, checksum-2 | A cumulative checksum over the header and every frame up to this one. | 

A frame is valid only if its salts match the header's and its cumulative checksum holds. A transaction is committed when a valid frame carries a nonzero commit size; frames after the last commit frame belong to a transaction that never finished.

### Readers and the wal-index

A reader starting a transaction notes the last valid commit frame as its end mark and ignores anything appended later, which gives it a stable snapshot. To find a page, it looks for the newest frame of that page at or before its end mark, and falls back to the database file if there is none. So that this lookup is not a scan, SQLite keeps a **wal-index** in shared memory, backed by the `-shm` file. Shared memory is also why a WAL database cannot live on a network filesystem: every process using it must be on the same host.

### Checkpoints

A **checkpoint** copies the committed frames back into the database file, syncs it, and lets the WAL start over from the beginning. By default SQLite runs a PASSIVE checkpoint whenever a commit leaves the WAL at 1,000 pages or more. The four modes differ in how hard they push:

| Mode | Behavior | 
|---|---|
| PASSIVE | Copies what it can without waiting for readers or writers. | 
| FULL | Waits for the writer and current readers, then copies every frame. | 
| RESTART | A FULL checkpoint that then waits for readers so the next writer starts the WAL from the top. | 
| TRUNCATE | A RESTART checkpoint that also truncates the `-wal`file to zero bytes. | 

One rule matters more than the rest for replication. A checkpoint can never overwrite database pages that an active reader still needs, so **a long-running read transaction stops the WAL from being reset**. Frames stay in the file until that reader lets go.

### The VFS

Everything SQLite does to files goes through a **VFS**, the operating-system interface at the bottom of the library: open, read, write, sync, lock, and the shared-memory calls behind the wal-index. A custom VFS, or a shim around the default one, can intercept any of them. That is how a database can be read from somewhere other than a local file.

## Litestream: replication from the WAL

Ben Johnson released **Litestream** in 2020 to give an ordinary SQLite application continuous backup to object storage, with no server to run. It is a separate process (or a Go library) beside an unmodified application, and it works entirely through the WAL rules above:

- It holds a **long-running read transaction**on the database. By the rule in § 02, SQLite then cannot reset the WAL under it, so no frame is lost before Litestream has copied it.
- It **takes over checkpointing**, running its own checkpoints when it chooses, after it has captured the frames.
- It copies each new run of committed frames, and uploads it to object storage.
- A **restore**downloads the newest snapshot, then replays everything written after it, in order.

Early versions stored raw WAL segments in a "shadow WAL", grouped into random-ID **generations** that started over whenever continuity broke. Restoring a busy database meant replaying every intermediate page write.

**Litestream v0.5** replaced that design. Johnson announced it in "Litestream Revamped" (2025·05·20), taking "our LiteFS learnings" back into Litestream, and v0.5.0 shipped on 2025·10·02. It brought:

- **LTX files**in place of WAL segments and generations, numbered by a monotonically increasing transaction ID (TXID).
- **Compaction levels**for fast point-in-time restore. By default L1 merges the L0 files of each 30-second window, L2 the L1 files of each 5 minutes, L3 the L2 files of each hour, and a full snapshot is taken daily, so "a dozen or so files on average" restore a database to any point.
- **A lease**built on object storage's conditional writes, so only one primary replicates to a destination.
- **Read replicas through a VFS**, which serve queries by fetching pages from object storage on demand.

## The LTX format

**LTX** began in **LiteFS**, Fly.io's distributed SQLite filesystem (introduced 2022·09·21), which needed a transaction-aware unit to ship between nodes instead of a raw WAL stream. An LTX file holds the pages a range of transactions changed, **sorted by page number**, with enough metadata to check that it applies cleanly.

| Part | Contents | 
|---|---|
| Header (100 bytes) | Magic `LTX1`, flags, page size, commit (database size in pages after the file), minimum and maximum TXID, a timestamp, the pre-apply checksum, the WAL offset, size, and salts it was read from, a node ID, reserved bytes. | 
| Pages | Each page: a small header (page number, flags), then the page data compressed with LZ4, in ascending page order, ended by an empty page header. | 
| Page index | For each page: its number, its byte offset in the file, and its encoded size. | 
| Trailer (16 bytes) | The post-apply checksum and a file checksum. | 

A file is named for its TXID range, `<min>-<max>.ltx`, in zero-padded hex. Three design choices do most of the work:

- **Sorted pages make files mergeable.**Two adjacent files can be- **compacted**into one that keeps only the newest version of each page, which is why restore needs few files.
- **Per-page compression plus the page index make pages addressable.**A reader can fetch and decode a single page with a ranged read, without downloading the file; this is what the VFS read replicas, and celld's paged restore, rely on.
- **Checksums make chains verifiable.**The- *file checksum*is a CRC-64 over the header, the page headers, the uncompressed page data, the index, and the post-apply checksum. The- *database checksum*is the XOR of a CRC of every page, so a transaction updates it incrementally; a file's pre-apply checksum must equal the database's checksum before it applies, and its post-apply checksum afterward. A file may also carry a- *no-checksum*flag that turns the database checksums off.

LTX has two page layouts. The original **frame layout** stores each page as an independent LZ4 frame. The **block layout** introduced in LTX v0.5.2 stores a 4-byte size and a raw LZ4 block, which is more compact. A reader that knows only the frame layout cannot read block files.

## How celld uses it

celld's replication library, `crates/ltx`, is a Rust port of this lineage. Its README records the provenance: it was seeded on 2026·08·03 from **rustyriver**, a from-scratch Rust reimplementation of Litestream v0.5 and LTX, and celld now owns it as first-class source. The replication behavior comes from Litestream v0.5.11 and the block format from Litestream v0.5.16 (LTX v0.5.2); a pinned port of the pierrec/lz4 compressor makes the compressed bytes match upstream exactly. The README also draws the boundary that the rest of this section follows: the library "captures committed WAL data as L0 LTX segments", while "the output gate, epoch fencing, replicated node log, and takeover recovery enforce the write acknowledgement contract".

### Capture

The library opens the cell's database on two connections. One holds a **long-running read transaction**, exactly as Litestream does, so the WAL cannot be reset before its frames are captured. The other runs the library's own checkpoints; auto-checkpoint is switched off on that connection, while the application's writer connection keeps SQLite's default, which is safe because the pinned read mark stops any reset. Every database also gets two small control tables, `_litestream_seq` and `_litestream_lock`, whose definitions match Litestream's character for character.

Capture happens only when the library's `sync()` is called: the crate has no timer of its own. In celld, the **output gate** drives it. A write that needs a durability proof takes a ticket, and a sync loop that wakes on that ticket or every 25 ms captures every cell with pending work. One `sync()` reads the WAL, keeps only frames that belong to committed transactions (salts and cumulative checksum valid, ending in a nonzero commit size), and writes **one new L0 file** at the next TXID. If the WAL no longer continues from the last capture (another process truncated it, or its salts reset), the library writes a whole-database file instead.

The library also decides when to checkpoint: a TRUNCATE once the WAL reaches a threshold (celld sets `CELLD_LTX_TRUNCATE_PAGES` to 128 pages), a PASSIVE after 1,000 new frames, and a time-based PASSIVE every 60 seconds. A Queue cell uses passive checkpoints only.

### Where the files go

Every object a cell produces lives under its **epoch prefix**, with the compaction level as a 4-digit hex directory:

```
<fleet prefix>cells/<cell>/ltx/e<epoch>/<level:04x>/<minTXID:016x>-<maxTXID:016x>.ltx
e.g. cells/chat-1/ltx/e7/0000/0000000000000005-0000000000000005.ltx   (an L0 capture)
```
The epoch in the key is the fence (Chapter 4): the uploads themselves are plain PUTs, and a stale owner can only write into its own, superseded epoch. This is why celld **removed Litestream's object-storage lease**. The port included it and deleted it on 2026·08·06, unused, because a lease file under the replica prefix "would be a second, competing layer".

Under fleet durability, captured segments reach the bucket in **bundles**: one object per flush interval that concatenates the L0 files of every dirty cell (magic `CLB1`), instead of one upload per cell. Bundles are celld's own addition, not part of Litestream. A cell's per-cell prefix is filled from them when it is needed, and "at rest, the bucket is pure Litestream".

### Compaction

celld runs **L1 only**, and **additively**: a compaction publishes a new L1 object covering a contiguous TXID run of L0 files and never deletes its sources. In each page, the newest version wins. A node-wide scheduler compacts a cell once 256 TXIDs or 32 MiB have accumulated, two cells at a time by default; the knobs are `CELLD_LTX_COMPACTION`, `CELLD_LTX_COMPACTION_MIN_TXIDS`, `CELLD_LTX_COMPACTION_MIN_MB`, and `CELLD_LTX_COMPACTIONS`. The payoff is the one Litestream's levels give: "a takeover reads tens of objects instead of thousands". celld does not produce Litestream's L2 or L3 tiers.

The two LTX layouts meet here. Ordinary L0 captures are still written in the older frame layout, while compaction output uses the v0.5.2 block layout. A node that cannot read block files could not restore a cell after the first L1 file appeared, which is why a mixed-version fleet must set `CELLD_LTX_COMPACTION=0` until every node can read them.

### How integrity is checked

celld does **not** rely on LTX's pre-apply and post-apply checksum chain. Every L0 capture, every compaction output, and the markers described below are written with the no-checksum flag. What holds a chain together instead is a **CRC-64 file checksum** on every file, checked on every full decode, and **contiguous TXID ranges**: restore refuses a chain with a gap, and refuses a file that grows the database without supplying every new page. The one checksummed file is the **handoff snapshot**, a full image published at level `0009` when a cell is released.

### Restore

A new owner rebuilds a cell one of two ways (Chapter 3).

- **Full restore.**Anchor on the newest handoff snapshot, then take the longest contiguous chain of files across the levels, verify each file's CRC as it decodes, and apply them oldest to newest, the newest version of each page winning. The result is written to a temporary file, synced, and renamed into place.
- **Paged restore**, for a chain of 256 MiB or more (- `CELLD_LTX_PAGED_MIN_MB`). celld builds a- **page map**from the page indexes at the tail of each file, then opens the database through a- **fault-in VFS**: the local file starts sparse, and the first read of a missing page performs one ranged read. The byte range is exact, taken from the page index, but a fault normally fetches a run of neighboring pages up to 1 MiB, and prefetches the children of an interior b-tree page on a sequential scan. A page that should exist but cannot be read fails the query rather than returning zeros. In the background, one cell per node at a time is- **hydrated**at- `CELLD_LTX_HYDRATE_MBPS`(16 MiB/s by default;- `0`keeps it sparse). This is why the store must serve exact ranged reads (Chapter 3). A paged fault cannot check a whole file's CRC, because it never reads the whole file; it checks the page header and the LZ4 decode.

A paged cell cannot afford to write a whole-database opener into its new epoch; on a 2 GB cell that took minutes through the fault path. So a paged epoch **continues the chain it paged from**: celld writes a zero-page **marker** file at the next TXID, and the next capture is a normal incremental file.

### The node log

Fleet durability adds one more piece outside the library (Chapter 3). Each node streams the L0 segments it has captured but not yet uploaded to a small ensemble of followers, and a write is acknowledged when every follower holds its segment on disk. When a node takes over a cell, it first seals the previous owner's log and uploads what the followers retained. The followers answer that recovery with a **log tail**: `CLT1` returns entries only, while `CLT2`, added in v0.6.0, also states the range it covers and whether it is complete. A v0.6.0 node that needs a follower's witness recovers its previous log session only from a `CLT2` tail, and a v0.5.1 follower can answer only with `CLT1`; that is the reason behind the v0.5.1 → v0.6.0 full stop under fleet durability (Appendix C).

## Where the book covers each piece

| Topic | Where | 
|---|---|
| LTX segments, bucket proof and fleet proof, the four store requirements, exact ranged reads | Chapter 3 | 
| Epochs, the ownership record, the output gate, self-fencing | Chapter 4 | 
| `CELLD_LTX_*`settings, compaction, and paged restore in operation | Chapter 5 | 
| Why a facet has its own SQLite file and replication stream | Chapter 6 | 
| The `CLT2`upgrade rule and the release history of paging and compaction | Appendix C | 

This chapter draws on: celld: documentation at v0.6.0 (4 entries) · SQLite's WAL, Litestream, and LTX (12 entries). The full entries are in the Bibliography.


---

<!-- https://tlockney.github.io/celld-book/bibliography.html -->

Every source this book draws on, grouped by subject. "Used in" names the parts of the book whose text cites the source; attribution is at that level, because that is how the sources were used. Pages were read between 2026·09·15 and 2026·09·28; celld's documentation is cited at the v0.6.0 tag so the links keep matching the text.

## celld: documentation at v0.6.0

- *celld documentation (main guide)*. denoland/celld, docs/README.md at v0.6.0. github.com/denoland/celld/blob/v0.6.0/docs/README.md — Used in: Chapters 2–8 and Appendices B–C; Chapter 9; Labs 1–3; Appendix D.
- *What celld guarantees*. denoland/celld, docs/guarantees.md at v0.6.0. github.com/denoland/celld/blob/v0.6.0/docs/guarantees.md — Used in: Chapters 2–8 and Appendices B–C; Chapter 9; Appendix D.
- *Cloudflare compatibility*. denoland/celld, docs/cloudflare-compat.md at v0.6.0. github.com/denoland/celld/blob/v0.6.0/docs/cloudflare-compat.md — Used in: Chapters 2–8 and Appendices B–C; Chapter 9.
- *Limitations*. denoland/celld, docs/limitations.md at v0.6.0. github.com/denoland/celld/blob/v0.6.0/docs/limitations.md — Used in: Chapters 2–8 and Appendices B–C.
- *Security*. denoland/celld, docs/security.md at v0.6.0. github.com/denoland/celld/blob/v0.6.0/docs/security.md — Used in: Chapters 2–8 and Appendices B–C.
- *Telemetry*. denoland/celld, docs/telemetry.md at v0.6.0. github.com/denoland/celld/blob/v0.6.0/docs/telemetry.md — Used in: Chapters 2–8 and Appendices B–C; Chapter 9.
- *Testing*. denoland/celld, docs/testing.md at v0.6.0. github.com/denoland/celld/blob/v0.6.0/docs/testing.md — Used in: Chapters 2–8 and Appendices B–C.
- *WebAssembly*. denoland/celld, docs/wasm.md at v0.6.0. github.com/denoland/celld/blob/v0.6.0/docs/wasm.md — Used in: Chapters 2–8 and Appendices B–C; Chapter 9.
- *Service pages: Workers, Durable Objects, Durable Object facets, static assets, cron triggers, Dynamic Workers, KV, Queues, D1, Workflows, R2, Containers*. denoland/celld, docs/services/ at v0.6.0 (also at celld.dev/docs/services). github.com/denoland/celld/tree/v0.6.0/docs/services — Used in: Chapters 2–8 and Appendices B–C; Chapter 9; Labs 1–3; Appendix A.
- *Example projects: counter, workflow, queues, facets, container, sandbox, d1*. denoland/celld, examples/ at v0.6.0. github.com/denoland/celld/tree/v0.6.0/examples — Used in: Chapter 9; Labs 1–3.
- *celld-ltx: README, source, and reference/ltx-format.md*. denoland/celld, crates/ltx at v0.6.0. github.com/denoland/celld/tree/v0.6.0/crates/ltx — Used in: Appendix D.
- *The replicated node log and its CLT1/CLT2 tail formats (crates/celld/node_log.rs, ltx_repl.rs)*. denoland/celld, crates/celld at v0.6.0. github.com/denoland/celld/tree/v0.6.0/crates/celld — Used in: Appendix D.

## celld: release notes

- *celld v0.4.0*. denoland/celld releases, 2026·08·28. github.com/denoland/celld/releases/tag/v0.4.0 — Used in: Chapters 2–8 and Appendices B–C.
- *celld v0.4.1*. denoland/celld releases, 2026·09·05. github.com/denoland/celld/releases/tag/v0.4.1 — Used in: Chapters 2–8 and Appendices B–C; Chapter 9.
- *celld v0.5.0*. denoland/celld releases, 2026·09·15. github.com/denoland/celld/releases/tag/v0.5.0 — Used in: Chapters 2–8 and Appendices B–C; Chapter 9.
- *celld v0.5.1*. denoland/celld releases, 2026·09·19. github.com/denoland/celld/releases/tag/v0.5.1 — Used in: Chapters 2–8 and Appendices B–C; Chapter 9.
- *celld v0.6.0*. denoland/celld releases, 2026·09·26. github.com/denoland/celld/releases/tag/v0.6.0 — Used in: Chapters 2–8 and Appendices B–C; Chapter 9.

## Cloudflare documentation

- *How Workers works*. Cloudflare Workers docs. developers.cloudflare.com/workers/reference/how-workers-works/ — Used in: Chapter 1.
- *Bindings (env)*. Cloudflare Workers docs. developers.cloudflare.com/workers/runtime-apis/bindings/ — Used in: Chapter 1.
- *What are Durable Objects?*. Cloudflare Durable Objects docs. developers.cloudflare.com/durable-objects/concepts/what-are-durable-objects/ — Used in: Chapter 1; Chapters 2–8 and Appendices B–C.
- *Use WebSockets*. Cloudflare Durable Objects docs. developers.cloudflare.com/durable-objects/best-practices/websockets/ — Used in: Chapter 1; Chapters 2–8 and Appendices B–C.
- *Alarms*. Cloudflare Durable Objects docs. developers.cloudflare.com/durable-objects/api/alarms/ — Used in: Chapter 1; Chapters 2–8 and Appendices B–C.
- *Rules of Durable Objects*. Cloudflare Durable Objects docs. developers.cloudflare.com/durable-objects/best-practices/rules-of-durable-objects/ — Used in: Chapter 1; Chapters 2–8 and Appendices B–C.
- *Overview*. Cloudflare Workflows docs. developers.cloudflare.com/workflows/ — Used in: Chapter 1.

## Cloudflare blog and talks

- Kenton Varda. *Workers Durable Objects Beta: A New Approach to Stateful Serverless*. The Cloudflare Blog, 2020·09·28. blog.cloudflare.com/introducing-workers-durable-objects/ — Used in: Chapter 1.
- Kenton Varda. *Durable Objects: Easy, Fast, Correct — Choose three*. The Cloudflare Blog, 2021·08·03. blog.cloudflare.com/durable-objects-easy-fast-correct-choose-three/ — Used in: Chapter 1.
- Greg McKeon. *Durable Objects — now Generally Available*. The Cloudflare Blog, 2021·11·15. blog.cloudflare.com/durable-objects-ga/ — Used in: Chapter 1.
- Matt Alonso and Greg McKeon. *Durable Objects Alarms — a wake-up call for your applications*. The Cloudflare Blog, 2022·05·11. blog.cloudflare.com/durable-objects-alarms/ — Used in: Chapter 1.
- Kenton Varda. *We've added JavaScript-native RPC to Cloudflare Workers*. The Cloudflare Blog, 2024·04·05. blog.cloudflare.com/javascript-native-rpc/ — Used in: Chapter 1.
- Kenton Varda. *Zero-latency SQLite storage in every Durable Object*. The Cloudflare Blog, 2024·09·26. blog.cloudflare.com/sqlite-in-durable-objects/ — Used in: Chapter 1.
- Sid Chatterjee, Matt Silverlock, and Celso Martinho. *Build durable applications on Cloudflare Workers: you write the Workflows, we take care of the rest*. The Cloudflare Blog, 2024·10·24. blog.cloudflare.com/building-workflows-durable-execution-on-workers/ — Used in: Chapter 1.
- *Sequential consistency without borders: How D1 implements global read replication*. The Cloudflare Blog, 2025·04·10. blog.cloudflare.com/d1-read-replication-beta/ — Used in: Chapter 1.
- *How Durable Objects and D1 Work: A Deep Dive with Cloudflare's Josh Howard*. YouTube (video). www.youtube.com/watch?v=C5-741uQPVU — Used in: Chapter 1.
- Boris Tane. *What even are Cloudflare Durable Objects?*. boristane.com, 2025·11·04. boristane.com/blog/what-are-cloudflare-durable-objects/ — Used in: Chapter 1.

## The actor model

- *Actor model*. Wikipedia. en.wikipedia.org/wiki/Actor_model — Used in: Chapter 1.
- *Erlang (programming language)*. Wikipedia. en.wikipedia.org/wiki/Erlang_(programming_language) — Used in: Chapter 1.
- *How the Actor Model Meets the Needs of Modern, Distributed Systems*. Akka core documentation. doc.akka.io/libraries/akka-core/current/typed/guide/actors-intro.html — Used in: Chapter 1.
- Phil Bernstein, Sergey Bykov, Alan Geller, Gabriel Kliot, and Jorgen Thelin. *Orleans: Distributed Virtual Actors for Programmability and Scalability*. Microsoft Research, technical report MSR-TR-2014-41, 2014·03. www.microsoft.com/en-us/research/publication/orleans-distributed-virtual-actors-for-programmability-and-scalability/ — Used in: Chapter 1.
- *Orleans overview*. Microsoft Learn. learn.microsoft.com/en-us/dotnet/orleans/overview — Used in: Chapter 1.
- David Khourshid. *On the actor model (post on X)*. X, 2026·03·15. x.com/DavidKPiano/status/2033132659795194367 — Used in: Chapter 1.

## Durable execution

- *The definitive guide to Durable Execution*. Temporal, 2025·05·06. temporal.io/blog/what-is-durable-execution — Used in: Chapter 1.
- *Key Concepts*. Restate documentation. docs.restate.dev/concepts/durable_execution — Used in: Chapter 1.
- *Durable Functions overview*. Microsoft Learn. learn.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-overview — Used in: Chapter 1.
- *Durable entities*. Microsoft Learn. learn.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-entities — Used in: Chapter 1.
- Jack Vanlightly. *Demystifying Determinism in Durable Execution*. jack-vanlightly.com, 2025·11·24. jack-vanlightly.com/blog/2025/11/24/demystifying-determinism-in-durable-execution — Used in: Chapter 1.
- Gunnar Morling. *Building a Durable Execution Engine With SQLite*. morling.dev, 2025·11·20. www.morling.dev/blog/building-durable-execution-engine-with-sqlite/ — Used in: Chapter 1.
- Peter Kraft and Qian Li. *Why Durable Execution Should Be Lightweight*. DBOS, 2025·01·12. www.dbos.dev/blog/what-is-lightweight-durable-execution — Used in: Chapter 1.
- *What's the Use Case for Durable Execution?*. DBOS, 2025·05·26. www.dbos.dev/blog/durable-execution-by-default — Used in: Chapter 1.
- Alex Poliakov. *The Superpowers of Durable Execution*. LinkedIn, 2025·11·06. www.linkedin.com/pulse/superpowers-durable-execution-alex-poliakov-xvk1e/ — Used in: Chapter 1.
- John Bellaud. *The Imperative of Durable Execution in App Dev: Unveiling Temporal's Framework*. TechFabric, 2024·10·23. www.techfabric.com/blog/the-imperative-of-durable-execution-in-app-dev-unveiling-temporals-framework — Used in: Chapter 1.

## SQLite's WAL, Litestream, and LTX

- *Write-Ahead Logging*. SQLite documentation. sqlite.org/wal.html — Used in: Appendix D.
- *Database File Format (the WAL file format)*. SQLite documentation. sqlite.org/fileformat2.html — Used in: Appendix D.
- *The SQLite OS Interface or "VFS"*. SQLite documentation. sqlite.org/vfs.html — Used in: Appendix D.
- *How it works*. Litestream documentation. litestream.io/how-it-works/ — Used in: Appendix D.
- *Command: ltx*. Litestream documentation. litestream.io/reference/ltx/ — Used in: Appendix D.
- *Migration Guide*. Litestream documentation. litestream.io/docs/migration/ — Used in: Appendix D.
- *Litestream: streaming replication for SQLite (README and docs/SQLITE_INTERNALS.md)*. benbjohnson/litestream. github.com/benbjohnson/litestream — Used in: Appendix D.
- *LTX: Go library for the LTX file format*. superfly/ltx. github.com/superfly/ltx — Used in: Appendix D.
- Ben Johnson. *Introducing LiteFS*. The Fly Blog, 2022·09·21. fly.io/blog/introducing-litefs/ — Used in: Appendix D.
- *How LiteFS Works*. Fly.io documentation. fly.io/docs/litefs/how-it-works/ — Used in: Appendix D.
- Ben Johnson. *Litestream Revamped*. The Fly Blog, 2025·05·20. fly.io/blog/litestream-revamped/ — Used in: Appendix D.
- Ben Johnson. *Litestream v0.5.0 is Here*. The Fly Blog, 2025·10·02. fly.io/blog/litestream-v050-is-here/ — Used in: Appendix D.


---

<!-- https://tlockney.github.io/celld-book/colophon.html -->

## How this book was made

This book was generated with AI, using three tools, from materials curated by Thomas Lockney for this purpose:

- **Writing.**The text, the diagrams, the self-quiz, and the lab notebooks were written by Anthropic's Claude, working in Claude Code.
- **Research.**Research on the core topics (the Workers platform, Durable Objects, the actor model, and durable execution) was done with Gemini Notebook, driven through the gemini-notebook-mcp-cli tool.
- **Review.**Reviews of the material were run with Hermes Agent, using the GLM-5.3 and DeepSeek V4.1 Flash models.

Thomas chose the subject and the sources, set the scope and the structure, and directed each revision, including acting on those reviews, which checked the text against celld's documentation and led to corrections. The sources were celld's documentation and release notes (v0.4.0 through v0.6.0), Cloudflare's documentation and engineering posts, and a curated set of articles on actors and durable execution. The Bibliography lists all of them.

The labs were not simulated. Each notebook was executed against a real `celld dev` node (celld v0.6.0, Deno 2.9.6) on a homelab JupyterLab on 2026·09·26, and the outputs are shown exactly as that run produced them. Where a run showed less than the prose once claimed, the prose was changed to match the output, never the reverse.

## Accuracy

The claims in this book were checked against the sources listed in the Bibliography, and many against a running node. Even so, AI-generated text can contain errors, and celld is a young project that changes quickly. Verify anything you depend on against the release you install and against celld's own documentation. This book is not affiliated with or endorsed by Deno or Cloudflare.

## History

| Date | Event | 
|---|---|
| 2026·08·30 | The field guide (now Part II) first written, for celld v0.4.0 | 
| 2026·08·31 | The step-by-step guide (now Part III) first written, for v0.4.0 | 
| 2026·09·15 to 09·26 | Revised for v0.4.1, v0.5.0, v0.5.1, and v0.6.0; the three labs written and executed | 
| 2026·09·27 | Rewritten as one description of v0.6.0; the primer (now Part I) and the review sheet added | 
| 2026·09·28 | Assembled as this book | 

## Production

The chapters are generated from the same markdown sources and notebooks as the companion Reading Room series, by a small Deno build. Pages are rendered with the "bench sheet" layout: section numbers, code languages, and callout kinds in the left margin, set in Source Serif 4, IBM Plex Sans Condensed, and JetBrains Mono. Code is highlighted with highlight.js and markdown is parsed with marked.
