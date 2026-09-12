# The Vietnam of Computer Science

Ted Neward's 2006 essay that permanently changed how the software industry talks about Object/Relational Mapping — by comparing it to America's most traumatic military quagmire. The essay diagnoses why ORM, despite early successes, inevitably becomes a source of compounding complexity, and catalogues the six fundamental problems that make the object-relational impedance mismatch unresolvable in principle.

---

## The Central Analogy

> "Object/Relational Mapping is the Vietnam of Computer Science. It represents a quagmire which starts well, gets more complicated as time passes, and before long entraps its users in a commitment that has no clear demarcation point, no clear win conditions, and no clear exit strategy."

Neward spends nearly a third of the essay on the Vietnam history before touching ORM. This isn't padding — it's structural. The history lesson establishes the pattern he's mapping: unclear initial goals, early successes that create commitment, the Law of Diminishing Returns as each escalation yields less, and the sunk-cost logic ("we've gone this far, surely we can see this thing through") that makes retreat feel like betrayal. The analogy was controversial from the start — comparing software architecture debates to a war that killed millions — but its longevity in the engineering lexicon suggests it captured something real.

## The Object-Relational Impedance Mismatch

The technical core of the essay is the claim that objects and relations are fundamentally different models, and "the two models are simply too different to bridge silently":

- **Objects**: identity (distinct from state), state, behavior, encapsulation → inheritance, polymorphism, unidirectional associations
- **Relations**: predicate logic, tuples as truth statements, attribute-based identity → set operators (restrict, project, join, etc.), bidirectional associations via foreign keys

> "Object systems are typically characterized by four basic components: identity, state, behavior and encapsulation. [...] Relational systems describe a form of knowledge storage and retrieval based on predicate logic and truth statements."

The gap isn't just syntactic (VARCHAR → String). It's semantic: objects model behavior and identity; relations model facts and truth. Bridging them requires mapping not just types but entire worldviews.

## The Six Problems

Neward catalogues six specific failure modes that any ORM must confront:

### 1. The Object-to-Table Mapping Problem (Inheritance)

> "Unfortunately, the relational model does not support any sort of polymorphism or IS-A kind of relation."

Three approaches, all broken: table-per-class (requires expensive JOINs across every class in the hierarchy for "find all Persons"), table-per-concrete-class (denormalization costs), or table-per-class-family (NULL columns everywhere, defeating integrity constraints). Each solves one problem by creating another.

### 2. The Schema-Ownership Conflict

> "At heart, many object-relational mapping tools assume that the schema is something that can be defined according to schemes that help optimize the O/R-M's queries against the relational data. But this belies a basic problem, that often the database schema itself is not under the direct control of developers."

The DBA vs. developer conflict isn't technical — it's political. But Neward correctly treats political problems as real problems: the schema will be frozen, report generators will need relational semantics, and the discriminator column you added for inheritance mapping will be "all but unusable" to Crystal Reports.

### 3. The Dual-Schema Problem

> "Updates or refactorings to one will likely require similar updates or refactorings to the other."

Metadata lives in two places (DDL and Java/C#). Refactoring the database requires data migration; refactoring the code doesn't — but O/R-M couples them. As the system grows, pressure builds to "tie off" the object model from the schema, which defeats the purpose of having an ORM.

### 4. Entity Identity Issues

> "If the two systems are going to agree on the sense of identity, the relational system must offer some kind of unique identity concept (usually an auto-incrementing integer column) to match that of the notion of object identity."

Objects have implicit identity (memory location). Relations have attribute-based identity (two identical rows are the same fact). ORMs paper over this with surrogate keys — but then caching, clustering, and concurrent sessions scatter identity across n+1 locations, none of which agree on what "the same object" means.

### 5. The Data Retrieval Mechanism Concern

Neward traces the QBE → QBA → QBL evolution as a series of escalating compromises:

> "Developers quickly note that the above approach is (generally) much more verbose than the traditional SQL approach, and certain styles of queries (particularly the more unconventional joins, such as outer joins) are much more difficult — if not impossible — to represent."

Query-by-Example forces domain objects to support nullable fields that violate domain rules. Query-by-API is verbose and vulnerable to "fat-finger" errors (string-based table/column names with no compile-time checking). Query-by-Language (HQL, OQL) is a subset of SQL that loses the "objects and only objects" selling point. Each step "solves" the previous problem while introducing new ones — the slippery slope in miniature.

### 6. The Partial-Object Problem and Load-Time Paradox

> "The problem here is that the data to be displayed in the first Display...() call is not the complete Person, but a subset of that data; here we face our first problem, in that an object-oriented system like C# or Java cannot return just 'parts' of an object."

SQL can `SELECT id, first_name, last_name` — return part of a relation. Objects can't return part of an object. The workaround (lazy loading) introduces the N+1 query problem and makes performance dependent on access patterns the ORM can't predict. "There will always be common use-cases where the decision made will be exactly the wrong thing to do."

## Six Possible Responses

> "Just as it's conceivable that the US could have achieved some measure of 'success' in Vietnam had it kept to a clear strategy and understood a more clear relationship between commitment and results, it's conceivable that the object/relational problem can be 'won' through careful and judicious application of a strategy that is clearly aware of its own limitations."

1. **Abandonment** — give up on objects entirely; return to procedural/relational programming
2. **Wholehearted acceptance** — give up on relational storage; use OODBMS (presciently noting "in an increasingly service-oriented world, which eschews the idea of direct data access... it becomes entirely feasible")
3. **Manual mapping** — write straight SQL and populate objects by hand; possibly code-generated
4. **Acceptance of limitations** — use ORM for 80% and raw SQL for the hard 20%; accept the caching risks
5. **Integration of relational concepts into languages** — bring sets into the language (pointing to LINQ, Scala, F# — all of which materialized after 2006)
6. **Integration of relational concepts into frameworks** — build domain frameworks around RowSets/DataSets rather than pure objects

Option 5 is the essay's most prescient prediction. Neward wrote this before LINQ shipped (2007), before Scala gained traction, before F# existed. He saw that the language itself was the right level to solve the impedance mismatch — not a library bolted on top. Two decades on, [[Acadia]] is the purest realization of option 5: Czaplicki's Elm-style language makes the schema a typed value, compiles `map`/`filter`/`select` to SQL at compile time, and answers the ORM question with "No objects!" — sidestepping Neward's six problems by never introducing objects in the first place.

> "Lash yourself to the mast if you wish to hear the song, but let the sailors row."

## Key Themes

- **#concept** Object-Relational Impedance Mismatch — the fundamental incompatibility between object and relational models that no tooling fully resolves
- **#pattern** The Slippery Slope — early ORM successes create commitment that makes retreat feel like invalidation of past investment
- **#concept** Sunk-cost fallacy in architecture — "we've gone this far, surely we can see this thing through" as a decision-making pathology
- **#pattern** Schema ownership — the political dimension of database design: who owns the schema and what happens when they disagree
- **#concept** Dual-schema problem — metadata in two places (code and DDL) that must stay synchronized under independent refactoring pressure
- **#concept** Partial-object / load-time paradox — objects can't be partially retrieved the way relations can, forcing lazy-loading compromises
- **#comparison** Six ORM strategies — from abandonment to language-level integration; the decision framework that outlasted the specific tools

## Critical Analysis

This essay matters because it named the problem correctly, before the industry was ready to hear it. 2006 was the peak of ORM enthusiasm: Hibernate was ascendant, Rails' ActiveRecord was spreading the "database as implementation detail" gospel, and Microsoft was about to ship LINQ to SQL and the Entity Framework. Neward's essay was a dissent from inside the establishment — he wasn't some NoSQL partisan (NoSQL didn't exist yet) but a working enterprise architect who had watched enough ORM projects fail.

The Vietnam analogy is the essay's strength and its weakness. It's memorable — the phrase entered the permanent lexicon — but it's also overheated. Comparing ORM bugs to 58,000 American dead is a category error that can distract from the technical argument. Neward anticipates this ("Recognizing that all analogies fail eventually") but went ahead anyway, and the essay works because the structural analogy (unclear goals → early success → escalating commitment → quagmire) is genuinely isomorphic to the ORM adoption pattern he describes.

The essay's technical catalog has aged remarkably well. The six problems he identified in 2006 are still the six problems ORM users wrestle with in 2026. What's changed is the industry's response: we've largely settled on option 4 (use ORM for the easy 80%, raw SQL for the rest) as the pragmatic default, and option 5 (language-level relational integration) has partially materialized through LINQ, language-integrated query in multiple ecosystems, and the resurgence of "just use SQL" as a respectable position.

For the agentic coding era, Neward's essay is newly relevant. [[Constraint Decay]] shows that LLM coding agents lose ~30 percentage points of assertion pass rate when databases and ORMs are added as constraints. The impedance mismatch isn't just hard for humans — it's hard for machines too, and for the same reasons. The dual-schema problem becomes a dual-context problem: the agent has to hold both the object model and the relational schema in its context window simultaneously, and they contradict each other.

The essay pairs naturally with [[The Cost YAGNI Was Never About]]: Beck's reframing of YAGNI as options pricing is the economic language for Neward's "slippery slope." Using an ORM consumes an option — the option to design your data access differently — and the cost of exercising that option compounds as the schema grows. It also resonates with [[Why Build vs Buy is the Wrong Question]]: ORM is a "buy" decision for data access, carrying all the integration-cost problems James identifies (mapping complexity, vendor upgrade cycles, the anticorruption layer you build around it).

---
*Sources: [[raw/031-01-neward-the-vietnam-of-computer-science-june-2006-pdf]], [[summary/031-01-neward-the-vietnam-of-computer-science-june-2006-pdf]]*
*Last updated: 2026-08-08*
