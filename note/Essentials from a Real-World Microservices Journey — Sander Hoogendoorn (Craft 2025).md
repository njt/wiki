# Essentials from a Real-World Microservices Journey — Sander Hoogendoorn (Craft 2025)

Sander Hoogendoorn's Craft 2025 talk is a war on dependencies fought from his day job as CTO of iBood, a Dutch e-commerce company run by 13 developers across ~150–160 repos deploying 40–50 times a day. The thesis: microservices aren't dead, but most teams adopt them for the wrong reasons — above all scalability — and the only justification that matters is destroying dependencies. Around that spine the talk assembles Gall's Law against big-bang rewrites, a uniform four-layer micro-architecture, DDD aggregates turned into services, micro frontends split along the same domain seams, snapshot-per-order data ownership, trunk-based development without pull requests, and Dee Hock's "simplify to amplify" as the management philosophy.

---

## What it argues

- **The wrong-reasons test.** "Most people solve the wrong problems with their microservices." Scalability is the canonical wrong reason — the insurance client's first service handled 68,000 logins per minute on a single JBoss instance; they never had a scalability problem. His closing corollary: if your microservices effort isn't working, you're probably doing it for the wrong reasons.
- **Dependencies kill; "technical death" follows.** Exhibit A: a public transport company's customer-service dependency graph and an SAP system with 142,000 tables. Exhibit B: iBood's pre-microservices platform, syncing data everywhere by every mechanism, which "totally broke down." Exhibit C: a landscape that took six weeks just to deploy. Companies in this state spend all their time keeping things up and die for lack of innovation.
- **Gall's Law beats the big-bang rewrite.** 18M lines of COBOL, 12M lines of Java, aging COBOL developers, six failed rewrite attempts at two years each. Microservices (adopted half-understood in 2014 — "it's new, it's cool") were the mechanism for growing small working systems instead.
- **Uniform, adaptable micro-architecture.** Every service has the same four layers — domain, repositories, use cases, resources — a pattern he's used since his Dutch UML book 25 years ago. Use cases double as the authorization unit. With 13 people and 160 repos, anyone must open any repo and know where everything is; hence the single TypeScript stack (mobile moving from Flutter to React Native).
- **Decompose with DDD.** Bounded contexts (Product means two things — replenishment vs. ordering — so split them), ubiquitous language, aggregates as the unit that becomes a service: a boundary, a root entity addressed by id mapping neatly onto REST, outside references only to the root. Boundaries aren't static — iBood's differ from two years ago. Cohesion is code that changes together, not technical layers.
- **Break up the front end too.** Services below and one app on top leaves the app holding all the business logic. iBood split the UI along the same seams (products, basket, checkout, account, orders); apps talk only to services, never to a database, so "my iBood" pages deploy without touching checkout.
- **Small releases are the point.** A bad release costs five minutes, not a quarter. Small deltas also make testing easier. The distrust spiral — bigger releases, longer intervals — ends with his annual-release anecdote: that company went bankrupt.
- **Automate everything; build quality in.** Check-in to production in ~30 minutes; pipeline definitions as shared code so a change propagates to all ~150 pipelines; SonarQube with 80% coverage measured on *new* code; Feathers' unit-test rules; trunk-based development with no pull requests or code reviews — in his context, with a concession for regulated 30-team organizations.
- **Services own their data.** Sharing a database just relocates the dependency graph. Orders snapshot everything at purchase time so a later price change doesn't mutate history. Document databases by default: an aggregate persists in one statement; relational only for hierarchical data.
- **Ways of working must change with the architecture.** No Scrum (quit 15 years ago), no sprints, no retrospectives, no Scrum master, everyone their own product owner, no estimates beyond ballpark. Event storming, pair/mob programming, self-organizing micro teams — "no two items that come off a board actually require the same set of people."

## Key quotes

> "Most people solve the wrong problems with their microservices."

Which, he notes as a former consultant, is "nice... you can get hired." The wrong-reasons test is the talk's most portable instrument — it converts an architecture decision into an audit question.

> "They suffer from what I call technical death, meaning they spend so much time keeping everything up that they have no room left for innovation anymore and they just get killed. Companies disappear because of that."

"Technical death" is the same diagnosis his companion talk [[Seven Habits of a Mostly Successful Team — Sander Hoogendoorn (Craft 2025)]] treats as an *organizational* disease — here it gets an *architectural* cause: dependencies.

> "A complex system designed from scratch never works and cannot be patched to make it work... you have to start over with a small working system, there's no way around."

Gall's Law, with the best evidence in the talk behind it: six failed two-year rewrites at the insurance company.

> "Missing the bus is not a big deal if you know there'll be another one along in five minutes."

The slide he tells the audience to remember ("I'm going to ask you tomorrow"). The whole release strategy — and its unpriced risk — is in this one metaphor: it presumes you know you missed the bus. See the criticism below.

> "Simple, clear purpose and principles give rise to complex and intelligent behavior... complex rules and regulations give rise to simple and stupid behavior... The trick is to simplify, to amplify."

Dee Hock, dramatized with Amsterdam removing traffic lights at a crossroads: take away the rules and drivers start making eye contact. This is [[The Forest and the Desert Are Parallel Universes]] operationalized at the level of process rules.

> "It's the first rule of microservices Club. You don't talk to others."

The Fight Club joke about services reaching into each other's databases — the memorable form of "a shared database just relocates the dependency graph."

> "I stopped using scrum 15 years ago. I don't like it very much. But that doesn't mean we do Kanban... There's like 50 shades of ways of working between."

Refusing the binary is characteristic: the talk's posture throughout is "what works for us might not work for you" — stated up front and repeated.

## Key themes

- #concept — technical death; the wrong-reasons test; Gall's Law; cohesion as "changes together"; simplify to amplify
- #pattern — aggregates-as-services; the four-layer micro-architecture (domain/repositories/use cases/resources); micro frontends along domain seams; snapshot-on-write for order history; trunk-based development; pipeline-as-shared-code
- #tool — SonarQube with coverage-on-new-code as a discipline enforcer; his open-source TypeScript microservices framework (architecture docs and examples on GitHub)
- #person — Sander Hoogendoorn, CTO of iBood, 47 years of programming, second talk of his in this wiki

## Critical take

The strongest material here is diagnostic, not prescriptive. The wrong-reasons test, Gall's Law with receipts, and "the database is still going to do this" (shared schemas relocating the dependency graph) are genuinely useful and testable. The snapshot-per-order pattern is correct and underused. And the uniform micro-architecture is an underrated forcing function: with 13 people and 160 repos, identical structure is what makes collective ownership physically possible.

But notice what every radical choice has in common: they all assume 13 senior people with a coding CTO. No pull requests, no code reviews, no estimates, no product owners, one stack, 160 tiny repos — this is a team-size configuration wearing a microservices costume. The digest is right that scaling the team is unanswered, and the no-PR argument actually indicts how reviews were done ("usually on the linting or the formatting" — which you can automate) rather than reviews as such. What catches design flaws now? Ad hoc pairing, apparently. [[Reviewing Code Is a Skill]] would want a word.

The acknowledged-then-dodged elephant is the monolith. He concedes the cohesion principle works in a modular monolith ("you should, by the way") and then never explains why iBood couldn't be one with the same CI/CD discipline. 160 repos for 13 people arguably argues the opposite of his thesis — that's an org chart, not physics. [[Designing Deploy-Time Flexibility for Modular Systems — Florin Coros, Craft 2025]], from the same conference, refuses the whole binary by making deployment a post-compile decision.

And the observability gap is the one that would actually hurt: the next-bus strategy depends on knowing you missed the bus, yet 150+ services are discussed with no monitoring, tracing, or alerting whatsoever. "A serious breakdown two days ago" appears only as a team-cohesion anecdote — nobody had to ask anyone to join the call — never as a diagnosis. Likewise unexplained: how business processes spanning services actually execute (everything appears to be synchronous HTTP), consistency for data that must be current across contexts, API versioning at 40–50 deploys a day, the cloud bill for 160 repos and ~150 pipelines, and security beyond use-case scopes at an e-commerce company handling payments. A coherent small-team story, honestly flagged by its own digest — and all the more credible for it.

## Related pages

- [[Seven Habits of a Mostly Successful Team — Sander Hoogendoorn (Craft 2025)]] — the same speaker, same conference, explicitly foreshadowed here ("I'm going to talk tomorrow a bit more about my team"): that talk is the governance half of this story (Tech Board, 70/20/10 budget, rule removal), this one the architecture half; they share every load-bearing fact — 13 people, 40–50 deploys a day, technical death, no PRs, microteams — and even the same live incident, which this talk calls "a serious breakdown two days ago" and that one calls the quarterly-sale outage he was still repairing.
- [[Microservices for the Benefits, Not the Hustle]] — strengthens Wolf's de-prioritization of scalability with a practitioner's receipt: 68,000 logins per minute on one JBoss instance as proof most companies don't have a scalability problem; both land on dependencies/changeability as the only honest justification.
- [[Designing Deploy-Time Flexibility for Modular Systems — Florin Coros, Craft 2025]] — complicates: Coros (same conference) makes "monolith or microservices" a category error and deploys the same binaries either way, while Hoogendoorn concedes cohesion applies to modular monoliths and then never answers why 160 tiny repos beat one; the two talks bracket the exact question the digest flags as dodged.
- [[The Forest and the Desert Are Parallel Universes]] — Hoogendoorn borrows Beck's keynote explicitly ("we're in the forest, not in the desert") and his Dee Hock / traffic-lights section is that metaphor made operational: remove the rules and people start making eye contact.

---
*Sources: [[raw/essentials-from-a-real-world-microservices-journey-sander-hoogendoorn-craft-2025]], [[summary/essentials-from-a-real-world-microservices-journey-sander-hoogendoorn-craft-2025]]*
*Last updated: 2026-09-13*
