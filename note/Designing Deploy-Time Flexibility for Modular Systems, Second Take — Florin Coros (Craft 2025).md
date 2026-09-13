# Designing Deploy-Time Flexibility for Modular Systems, Second Take — Florin Coros (Craft 2025)

A second ytx digest of Florin Coros's Craft 2025 talk on deploy-time flexibility — transcript byte-identical to the first ingest, digest regenerated with a sharper structure. The talk's argument is unchanged: "monolith or microservices" is the wrong question; build a modular system (dependencies only through contracts, per-contract proxies, attribute-driven type discovery via App Boot) and decide in-process versus distributed *after* compiling, by copying binaries between deployment folders. This page records what the second digest generation adds: a itemized mechanics catalog and a twelve-item audit of what the talk leaves out.

---

## Two digests, one talk

The transcript here is byte-identical to `raw/3bb805c5ca7cd00d365ee18d23cdb38a`; only the digest differs. The first digest led with "decide after compiling, not while designing"; this one leads with "decompose first; decide monolith-vs-microservices at deploy time" — a framing that is arguably truer to the talk's causal order, since Coros's whole point is that decomposition must be settled on contract terms before the deployment question even becomes askable. Two independent digest generations converging on the same core claims (three pillars, the bank story, the ~10-services cost curve) is decent evidence the talk's content is stable; the divergences are in emphasis, not substance.

## Key quotes

> "With monoliths, it's always all or nothing."

The hinge his whole mechanism turns on: you cannot scale, modernize, or test one piece of a monolith in isolation. Worth noting that this is also, read carefully, the indictment of his own co-location fix — merging services back into one process is a partial return to "all or nothing."

> "One of them opens a screen which has a bug, NullPointerException. That thing will blow up the entire system. All the users will be affected."

The blast-radius argument against monoliths, staged with 50 concurrent users. This digest is the one that notices the problem: co-locating chatty services reintroduces exactly this failure mode. Local proxies translate errors; they do not contain crashes.

> "They have spent three years to split the monolith. And now the resulting system was not working correctly."

The bank story's punchline — the decomposition succeeded and the system failed anyway, because 10–15-hop call chains accumulated network latency until "place an order" was unacceptable. Three years of correct decomposition defeated by a deployment decision nobody revisited.

> "It's basically like a DNS for production."

His one-line definition of service discovery: a registry mapping service names to addresses, changed when topology changes instead of chasing configs. The demo used hardcoded addresses.

> "Nobody knows how much it will take to build it. How much will it cost? It's astronomical."

Used twice, symmetrically — once for one giant service, once for a thousand tiny ones. The digest's best catch: this symmetric ignorance *is* the argument for the flat bottom of the build/integrate cost curve, and it is the only number-adjacent honesty in a talk motivated by latency that contains no benchmark anywhere.

## What this digest adds

The Tools section itemizes the mechanism the first digest summarized loosely. App Boot works by attribute: `[Service]` declares an implementation; `[ServiceProxy]` registers proxies at lower priority — "always the last ones will override the first ones" — so the invariant is that proxies are always deployed (every call *can* go over the wire) and a real implementation found in the deployment folder overrides its proxy, turning the call in-process. The Bootstrapper abstracts the DI container itself (Unity in the demo, pluggable), which is what keeps the host generic. Contract assemblies get a concrete DTO rule: "simple and concrete types… an array rather than an IList." And on enforcing zero module-to-module references he names three options — ReSharper-generated dependency graphs, Visual Studio Enterprise architecture validation, reflection-based reference tests in CI — then admits he mostly skips automation because illegal references are "very visible" in code review, with `internal` implementations as tooling-level friction. Production plumbing appears too: a generic host either convention-driven or fronted by dummy proxies that 404 un-deployed contracts, one Dockerfile per deployment configuration, co-location pairs chosen from production metrics, and DLL preloading where startup latency matters (desktop frontends; servers just treat scanning as warm-up).

The Unanswered Questions section is the sharpest artifact — twelve omissions, several of them failure modes the mechanism *creates* rather than merely ignores: **in-process boundary erosion** (REST's serialization boundary enforces encapsulation between services; in-process function calls pass shared object references, so mutable DTOs crossing module boundaries — the classic modular-monolith hazard — go unmentioned); the **testing matrix** (if topology is a free variable, does every configuration need testing?); the **bank story's unreported outcome** (he won the project and staffed ~20 people, but never says whether co-deployment actually fixed the latency); **AOT/trimming** unweighed against a reflection-heavy design; **contract versioning** across independently deployed services; the **honor-system "no logic" rule** (dependency rules get CI enforcement options; "no ifs, no fors" in contract assemblies gets none); and **topology governance** reduced to one passing mention of metrics.

## Key themes

#person Florin Coros · #concept modular system, contract-only dependencies, deploy-time flexibility · #pattern deferred deployment decision, proxy-priority injection, generic host · #tool App Boot (iquark/appboot)

## The take

The first take's verdict stands: a strong structural argument wrapped around an uncosted mechanism. What this digest changes is the *precision* of the bill. The first note priced the mechanism abstractly ("dual proxy sets, a custom discovery library, registration-behavior priority ordering"); this one itemizes it — attribute plumbing on every implementation, a pluggable DI abstraction, build artifacts per deployment configuration, a discovery registry, warm-up machinery — and adds the failure modes the first note missed, of which boundary erosion is the most serious, because it means the in-process mode is not simply "the same system, faster": it is a *weaker* contract regime, and the flexibility being sold is partly the option to weaken your own encapsulation.

The digest also makes explicit what the first note had to infer: the two elephants are fault isolation and independent scaling, and they are not peripheral — they are the very benefits Coros cited for leaving monoliths. The honest reading is that deploy-time flexibility doesn't resolve the monolith/microservices trade-off; it converts it into a per-service-pair decision that now needs data ("metrics from production"), governance, and rollback — none of which the talk supplies. What survives is unchanged and worth keeping: contract-only dependencies enforced by structure, the ~10-service sizing heuristic, and the meta-diagnosis that the industry cycles between architectures because it asks the wrong question. For the agent-era coda — that contract-shaped codebases are legible to coding agents too — see the first take, which lands it via [[The Economic Benefit of Refactoring]].

## Relate

- [[Designing Deploy-Time Flexibility for Modular Systems — Florin Coros, Craft 2025]] — the first ingest of this same talk; this second digest strengthens its critique (the elephants named outright, the bank's missing epilogue) and replaces its abstract cost-of-mechanism complaint with an itemized one.
- [[Microservices for the Benefits, Not the Hustle]] — Sander Hoogendoorn's same-conference claim that the only honest justification for microservices is destroying dependencies; this digest's elephants complicate Coros's co-location cure in exactly the way they complicate Hoogendoorn's scalability rationale — both talks want the decomposition without paying the distribution.
- [[Evolutionary Architecture — Maciej Jedrzejewski (Craft Budapest)]] — Jedrzejewski defers the same decisions up a ladder ("leverage what you have," extract only when teams step on toes) while Coros builds the flexibility in advance; the two are the same deference principle implemented at different layers, and both digests share the identical blind spot: data, transactions, and consistency are never addressed.

---
*Sources: [[raw/1f92f48318eb627c1f789e6c619ab9e1]], [[summary/1f92f48318eb627c1f789e6c619ab9e1]]*
*Last updated: 2026-09-13*
