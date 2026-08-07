# Event Sourcing — Set-and-Remove Bi-Temporal Events

Part twelve of Urs Enzler's event sourcing series distinguishes two kinds of bi-temporal event streams: lifetime events (create-update-delete) where events represent a thing's entire existence, and set-and-remove events where the stream is a timeline of "which data is relevant at any given time." The key technical contribution is the override-vs-insert choice: when a late-arriving event has an earlier effective date than an existing one, the system must decide whether the newcomer *overrides* everything after it or *inserts* between existing events. Both are valid, and the projection behavior must be explicitly configurable per stream.

---

## Key Quotes

> "a timeline which specifies which data is relevant at any given time"

This is the core distinction. Lifetime events answer "what state is this thing in?" while set-and-remove events answer "what data applies right now?" The shift is subtle but changes everything about how you project.

> "either a later-added event with an earlier effective date is inserted or overrides later events"

The article's nameable contribution. Bi-temporal systems have always wrestled with late-arriving data, but Enzler frames it as a deliberate, per-stream configuration choice rather than an edge case — and gives it two names with clear semantics. That's good API design thinking.

> "both scenarios can be valid"

The line that saves the article from dogmatism. Enzler resists the temptation to pick a winner, instead insisting the system expose the choice. This is mature engineering: the framework provides the knob, the domain expert turns it.

---

## Key Themes

- **#pattern** — Bi-temporal event sourcing with explicit override/insert semantics per stream
- **#concept** — Effective time vs. application time as orthogonal axes in event streams
- **#tool** — .NET/F# event sourcing with `ProjectionActionConfiguration = SetRemove`

---

## Critical Analysis

**What's valuable:** Enzler names something practitioners already do implicitly. Every event-sourced system with late-arriving data has an override-or-insert policy — it's just usually buried in a projector's sort logic, invisible and untested. Pulling it into configuration surface is the kind of design move that separates frameworks from ad-hoc implementations.

**What's underbaked:** The article gestures at `Sets (effective, value)` and `Removes effective` as though they're obvious primitives, but they carry a lot of unstated semantics. What happens when you remove something that was never set? What if two set events arrive with the same effective timestamp? The "both scenarios can be valid" line is honest but also a punt — it hands the complexity to the user without helping them decide. A decision tree or diagnostic ("you want override when X, insert when Y") would be more useful than a shrug.

**The big picture:** This is infrastructure writing at its best — not flashy, not revolutionary, but the kind of article that saves someone six months of getting it wrong in production. The set-and-remove pattern appears everywhere (rosters, feature flags, pricing rules, policy assignments) and most teams reinvent it badly. Enzler is doing the slow work of building a shared vocabulary for temporal data modeling, and that compounds.

**The operational side.** Enzler's focus is modeling — how to structure events correctly. The complementary question is what to do when those events turn out to be wrong. Oskar Dudycz's [[Fixing Bugs in Event Sourcing is Hard]] covers that ground: the same hotel reservation bug, walked through both architectures. His corrective-event pattern (append `ReservationPriceCorrected`, don't edit the original) and `buildSha` metadata trick (query by the commit that produced the event) are the operational practices that make bi-temporal modeling worth the investment.

**An RDF-native variant:** The [[Linked Data Event Streams (LDES)]] specification formalizes this same pattern — append-only event stream with version semantics — for the Semantic Web stack. LDES defines explicit predicates for create/update/delete objects and separates chronological order (`ldes:timestampPath`) from version order (`ldes:versionTimestampPath`), the same effective-time-vs-publication-time distinction Enzler draws. The key difference is that LDES targets HTTP-based data publishing rather than in-process projections, adding a synchronization algorithm, retention policies, and hypermedia traversal on top of the event sourcing substrate.

---

*Sources: [[raw/event-sourcing-set-remove-bi-temporal-events]]*
*Last updated: 2026-07-18*
