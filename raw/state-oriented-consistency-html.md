---
url: https://keel-iot.eu/blog/state-oriented-consistency.html
date_fetched: 2026-08-07
---

## Every distributed system has different kinds of state

The mistake we made, early on, was asking the wrong question.

We kept asking: **"Which consistency model should the cluster use?"**

The useful question turned out to be: **"Which consistency guarantee does this specific piece of state actually need?"**

Those sound similar. They aren't. The first one assumes a single answer exists for the whole system. The second one assumes it doesn't — and it's the second one that this whole piece converges on, in the form of one table that, by the end, does most of the explaining for us.

We discovered the difference the hard way while building a clustered message broker. The lesson underneath has very little to do with MQTT. Any distributed system holding more than one kind of state runs into this question eventually. Get it right, and the rest of the design gets simpler almost automatically.

## The incident

Two pods, five minutes apart, killed by the kernel's OOM handler. Memory limit: 512Mi. Nothing exotic — a normal container limit for a normal stateless service.

The first hypothesis was the obvious one: load-balancer imbalance, probably compounding into a reconnect cascade. Plausible. Wrong.

Active connections, three pods, same window:

Pod A: 8 ██ Pod B: 174 ████████████████ Pod C: 1,039 ████████████████████████████████████████

Working-set memory, same three pods, same window:

Pod A: ~270MB ████████████████████████████ Pod B: ~310MB ████████████████████████████████ Pod C: ~360MB █████████████████████████████████████

That's not the shape imbalance produces. A pod serving 130x more connections than another should not have nearly identical memory. Either the metric was lying, or the mental model was.

## The investigation

The metric wasn't lying.

Reading the actual code path responsible for session state turned up the real cause: on every pod boot, a persistence hook loaded *every client's* stored session state — the entire fleet's, not just the fraction that would ever reconnect to this specific pod. One unfiltered read, called once at startup, and every row it returned became a live in-memory object.

Why would anyone write it that way? In a stateless cluster behind a non-sticky load balancer, there's genuinely no way to know in advance *which* clients will reconnect to *this* pod. So the simplest correct-seeming implementation was: load everything, everywhere, and let whatever connects find its state waiting. Not a typo-bug — a design that quietly assumed a node should be ready to serve any client that might show up. A small, local instance of exactly the wrong question from the opening: it optimized for "the cluster can serve anyone," not "this specific piece of state has this specific requirement."

The chain rarely announces itself as an architecture problem:

```
Wrong abstraction
        ↓
Wrong guarantee
        ↓
Wrong architecture
        ↓
Wrong scaling
```
    Wrong abstraction: treating "who serves this client" as if every node needed the answer, not just one. Wrong guarantee: replicating everywhere, "just in case." Wrong architecture: no boundary on what a node is responsible for. Wrong scaling: cost grows with fleet size, not with actual responsibility.

An OOM is just where the chain happened to become visible. It could just as easily have surfaced as a slow memory leak, a scaling limit nobody could explain, or a rolling update that mysteriously got riskier as the fleet grew.

## The question this surfaces

Stated plainly, it stops looking like a memory bug and starts looking like a modeling error:

**We had assumed this state needed to be replicated everywhere any client might land. It didn't need that at all — it needed exactly one owner, decided by a rule any node could compute on its own.**

That's the question that outlives this one incident: not "how do we fit this in less memory," but "what does this state actually require, and have we been giving it more than that?"

## Naming the mistake, before fixing it

The default we'd fallen into deserves a name too, even though nobody chose it on purpose. We started referring to this design habit as **Uniform Consistency**: apply one consistency strategy to the whole system, and let every piece of state inherit it, regardless of what that specific state actually needs.

To be clear, we don't mean a specific algorithm — Uniform Consistency isn't a technique you'd find in a paper. We mean the habit: reaching for whatever consistency model the system already trusts, everywhere, by default, without asking each piece of state whether it actually needed that much.

```
 Wrong assumption                    Reality
 ────────────────                    ───────
 Every piece of                      Each piece of state
 distributed state                   has its own semantics —
 needs the same                      its own answer to
 guarantee.               ───────►   "what happens if two
                                      nodes briefly disagree?"
```
    Nobody designs a system by deciding "we will apply Uniform Consistency." It's what happens by default, one small decision at a time, when the question at the very start stays unasked. It's not a strawman — it's the thing we were actually doing, and the OOM is what it costs when you keep doing it long enough.

## Classifying the state

Asking "what does *this* state actually require" of every piece of cluster state — not just the one that had just caused an outage — is where the table comes from. Everything else in this piece is really just applying one reframe.

Consistency is not a property of a system.

It is a property of individual pieces of state.

| State | If two nodes briefly disagree... | Minimum sufficient guarantee | 
|---|---|---|
| Live connection ownership | Duplicate delivery, or a ghost connection nobody's driving | Single authoritative owner, decided right now | 
| Offline client ownership | Nothing bad — no live connection to duplicate | Deterministic — computable, no arbitration needed | 
| Message routing | A briefly stale route, self-correcting shortly after | Available — tolerant of brief staleness | 
| Durable message state | Silent, permanent data loss | Durable — no exceptions | 
| Cluster membership | A node briefly looks alive when it isn't | Eventually consistent | 

Five rows, five different minimum guarantees. The insight isn't any single row — it's that a system built by capable people will still default to Uniform Consistency unless something forces this question onto the table, row by row.

### The framework

This is the part we think generalizes past MQTT, past distributed brokers, past this project entirely:

```
   1. Identify the state.
             │
             ▼
   2. Ask what happens if two nodes disagree.
             │
             ▼
   3. Find the minimum guarantee that's actually required.
             │
             ▼
   4. Choose the weakest mechanism that provides that guarantee.
             │
             ▼
   5. Repeat, for the next piece of state.
```
    The exact answers will differ by system. A scheduler, a database, a service mesh, and an MQTT broker won't classify the same state the same way — a scheduler's "who owns this task" isn't a broker's "who owns this session," and shouldn't be forced to be. The loop stays the same. Only the rows in the table change.

The point isn't that different systems arrive at the same answers. The point is that they can arrive at them using the same reasoning process.

We started referring to this design rule internally as **State-Oriented Consistency** — not because it's a new distributed-systems theory, but because giving the idea a name made it easier to apply consistently across the project, and easier to catch ourselves the next time we reached for Uniform Consistency out of habit. We don't claim the idea is new. We found that giving it a name made it easier to reason about consistently.

Once we started classifying state this way, the architecture almost designed itself. Each row naturally suggested a different mechanism, because each row required a different guarantee.

## What we actually built

Semantics determine guarantees.

Guarantees determine mechanisms.

Mechanisms shape the architecture.

That table translated into one mechanism per row, each independent of the others — and the mapping only makes sense read as three columns, not two, because the guarantee is what decides the mechanism, not the other way around:

```
   STATE                    GUARANTEE                MECHANISM
   ─────                    ─────────                ─────────
   Live sessions            Single authoritative      Coordinator
                             owner
   Offline sessions         Deterministic              Deterministic
                                                        placement
   Topic routing            Availability                AP routing
   Durable message state    Durability                  Durable shared
                                                          storage
   Membership               Eventual consistency        Membership
                                                          gossip
```
    (The specific implementations behind each mechanism — which coordinator, which hashing scheme, which store — show up later in this piece and in the technical posts linked at the end. The diagram deliberately doesn't name them, because the decision that matters here is the guarantee each row needed, not which library ended up satisfying it.)

Once the guarantees were identified, selecting mechanisms became almost mechanical — the implementation simply followed the classification.

A small coordinator for the one row that truly needs an immediate, arbitrated answer. A deterministic scheme for state that only needs *an* answer, not an arbitrated one. A store optimized for availability where staleness is cheap. Durable shared storage for the one row where losing data isn't an option, no matter how available or fast the rest of the system is. Gossip for membership. Each row mapped to its own mechanism, none of them doing another's job.

People often ask which consensus algorithm we chose. That turned out to be the least interesting architectural decision we made. It's a solved problem — pick a mature implementation, and it does exactly the job a coordinator is for. The decision that actually mattered, and the one that's easy to skip past, was identifying which parts of the system deserved consensus *in the first place* — and, just as importantly, refusing to let that mechanism creep into the rows that didn't need it, just because it was already sitting there and already trusted.

We also drew one firm line: this coordination layer never carries bulk message traffic, in either direction. A coordinator that also carries payload caps your throughput at consensus latency — the opposite of what a system moving real traffic needs.

## The redesign

```
BEFORE: Fleet responsibility            AFTER: Derived responsibility
─────────────────────────               ──────────────────────────
 Every node                              Membership changes
    │                                          │
    ▼                                          ▼
 is responsible for                        ownership is computed
 the whole fleet                           deterministically —
    │                                      "this node owns X"
    ▼                                          │
 load everything,                              ▼
 no filter                                 materialized as a
    │                                      derived, rebuildable index
    ▼                                          │
 ~50-65% of memory limit                       ▼
 spent before serving a                    pod boots, loads X —
 single real connection                    only what it now owns
    │                                          │
    ▼                                          ▼
 OOMKilled under real load                cost scales with
                                           responsibility, not
                                           fleet size
```
    It's tempting to describe this as "we fixed an OOM." That undersells it.

**We weren't fixing an OOM.**

**We were removing an architectural assumption** — that any node needed to be ready to serve any client, which is what made "load everything, everywhere" look like the simplest correct implementation in the first place.

## The result, as properties rather than a single number

| Before | After | |
|---|---|---|
| Memory cost per node | Scales with the whole fleet | Scales with what that node owns | 
| What loads at boot | Everything, unconditionally | Only what ownership says belongs here | 
| Where the "truth" lives | A replicated copy on every node | A derived index — lost, it's recomputed, not restored | 

The floor that consumed most of the memory budget before serving a single real connection didn't get smaller through tuning. It disappeared, because the thing being loaded stopped scaling with something it never needed to scale with.

The implementation followed the classification, not the other way around.

## What we actually learned

We initially thought this state needed consensus, because "who owns this" sounds, on the surface, exactly like the kind of question consensus answers. It didn't. It needed a deterministic guarantee — an answer any node could compute alone, because there was never actually a race to arbitrate in the first place.

The architecture became simpler not because it had fewer mechanisms.

It became simpler because every mechanism had exactly one job.

We stopped asking how to make every piece of state strongly consistent.

We started asking what each piece of state actually needed.

Once the guarantees were clear, the mechanisms almost chose themselves.

Distributed systems rarely need one consistency model. They need the right one, applied one piece of state at a time.

The hardest part wasn't choosing better algorithms.

It was learning to ask a better question.

This architecture is implemented in

Keel MQTT Gateway, an open-source clustered MQTT broker written in Go. It currently runs in production after replacing a previous clustered MQTT broker, and has already survived rolling upgrades and a Kubernetes control-plane upgrade — the redesign above was proven by that replacement, not by a benchmark.

## Further reading

We deliberately kept the closer calls out of this piece — they're each their own story, not a footnote to this one.

**Coming next:** *Why We Didn't Use CRDTs* — the fuller case, including where CRDTs genuinely do earn their complexity, and where the line sits.

(Link added once it's published.)
