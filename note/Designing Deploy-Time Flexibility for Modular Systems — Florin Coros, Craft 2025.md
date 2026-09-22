# Designing Deploy-Time Flexibility for Modular Systems — Florin Coros, Craft 2025

Florin Coros's Craft 2025 talk (Budapest, transcribed from YouTube into a ytx gist) argues that "monolith or microservices" is a category error: the thing worth building is a *modular system*, and deployment should be a decision made after compiling, not while designing. Using contract-only dependencies, per-contract proxies, and attribute-driven type discovery, the same binaries run in-process as a modular monolith or distributed as microservices — his demo flips between the two by copying DLLs between folders. Born from a Dutch investment bank whose three-year monolith split produced 10–15-hop call chains too slow to place an order, the talk is equal parts architecture argument, .NET mechanics demo, and a candid (if incomplete) account of what such flexibility costs.

---

## Key quotes

> "Microservices or monoliths is not the right question to ask. What I think you all want to build is a modular system."

The thesis, delivered after cataloguing how both architectures fail at the same thing. Coros's move is to split two decisions everyone bundles: *how the code is structured* (modularity, contracts, decomposition) and *how it is deployed* (processes, network, latency). Bundle them and you get architecture debates that are really deployment debates in disguise — which is why the industry swings between the two poles every decade.

> "The architect has to decompose the system and to be in the area of minimum cost and to keep it there. Nothing else really matters. That's all it has to do."

His redefinition of the architect's job via the two-cost graph: cost to build services falls as you consolidate, cost to integrate them rises as you fragment. The flat bottom of that curve sits near ~10 services in orders of magnitude — and because the curve has diminishing returns, "don't chase the exact minimum" is the operative advice. It's a deliberately reductive job description; its virtue is that it makes sizing a first-class architectural act rather than an afterthought.

> "In this place, we can only write contracts. There is forbidden to write any logic… no ifs, no fors, no boolean expressions, no logical operators, nothing. Just interfaces."

Contract assemblies as a *physical* boundary, not a convention: interfaces, DTOs, fault exceptions, zero logic, enforced by project structure so tooling won't even suggest reaching past it (implementations are `internal` on purpose). This is the load-bearing pillar — the deploy-time flexibility is a party trick; the contract discipline is what survives regardless of whether you ever flip the switch.

> "Communication shouldn't be our primary concern… that should be something secondary, like a different thing we are going to approach at a later stage."

The root-cause claim for why both monoliths and microservices end up badly decomposed: decomposition gets done while subconsciously asking "will these two things live on the same server?" Answer that question at design time and your boundaries inherit a deployment answer they may have to outlive.

> "It's harder to convince a manager that you want to invest in your design to make it reusable. Because what the manager hears is that someone else, another project, will benefit of our work."

Why the stability triad (maintainability, extensibility, reusability — "You cannot have one without the other") gets under-invested: reusability's payoff is off-budget-sheet. Engineering-wise, extensibility and reusability are what *buy* the maintainability teams actually ask for.

> "So the contract is not consistent if you switch the types of deployment and this is not good."

His most production-honest moment: in the demo, in-process calls hit real implementations, so a bug surfaces as a raw IndexOutOfBoundsException where a distributed deployment would yield a 500. Production therefore needs a second set of in-process proxies wrapping real implementations for consistent error handling, logging, and telemetry — "You should always use proxies for this work in production. Actually this is how we do it in production, but for the demo I just took the shorter route."

## Key themes

#person Florin Coros · #concept modular system, contract-only dependencies, the build/integrate cost curve · #pattern deploy-time flexibility, deferred deployment decision, generic host · #tool App Boot (iquark/appboot)

## The take

The genuinely durable idea here is *deployment as a deferred, reversible decision* — the last-responsible-moment principle applied to architecture, with contracts as the hedge that makes deferral safe. The demo (copying DLLs between folders to flip in-process ↔ REST) is theater, but honest theater: Coros repeatedly distinguishes what the demo does from what production requires, and his digest of unanswered questions is more rigorous than most talks' Q&A.

The central tension goes unconfronted, though. Co-locating chatty services for performance gives up exactly the independent scalability that motivated splitting the bank's monolith in the first place — the fix partially undoes the treatment. And for a talk motivated by network latency, there are no numbers anywhere: no benchmarks, no measurement story for even *finding* the chatty pairs ("look through some metrics from production"). The mechanism's own bill is never weighed either — dual proxy sets, a custom discovery library, registration-behavior priority ordering, and a runtime whose behavior depends on deployment layout all cost real onboarding and debugging effort, none of it priced against just picking one deployment style. Add the silence on data, transactions, security, and tracing, and the honest summary is: a strong structural argument wrapped around an uncosted mechanism.

What survives scrutiny is worth keeping. Contract-only dependencies enforced by structure is good architecture even if you never exercise the flexibility; the ~10-service cost curve is a useful sizing heuristic; and the meta-diagnosis — the field cycles between architectures because it asks the wrong question — is the line that lasts. One more observation from this wiki's vantage, not his: in an agent era where [[The Economic Benefit of Refactoring]] shows semantic decomposition cutting an agent's token bill by 83%, modular structure pays twice — Coros's contracts would make a codebase legible to humans *and* to agents, even if the deploy switch never gets flipped.

## Relate

- [[Microservices for the Benefits, Not the Hustle]] — Oliver Wolf defends microservices on changeability rather than scalability; Coros complicates that defense by attacking the binary itself — and by leaving the one genuine microservices payoff (independent scaling) unaddressed in his own design.
- [[Architecture Is Designing Knowledge Flow]] — Diana Montalion's same-conference argument that coupling starts in mental models ("our microservices are tightly coupled because our brains were"); Coros's "we don't ask the correct question" is the mechanical twin of her cognitive diagnosis.
- [[Your Distributed System Is Slower Than a Laptop]] — the bank's 10–15-hop place-an-order chain is a live instance of the distribution tax COST quantifies; Coros's co-location fix is the same medicine (collapse the network) applied inside an enterprise split.
- [[The Wrong Abstraction]] — Sandi Metz prices the wrong abstraction; Coros names the wrong *axis* of decomposition (servers instead of contracts) as how you end up "here or here without knowing it" on the cost curve.

---
*Sources: [[raw/designing-deploy-time-flexibility-for-modular-systems-florin-coros-craft-2025-2]], [[summary/designing-deploy-time-flexibility-for-modular-systems-florin-coros-craft-2025-2]]*
*Last updated: 2026-09-13*
