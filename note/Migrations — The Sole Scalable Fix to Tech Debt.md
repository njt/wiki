# Migrations — The Sole Scalable Fix to Tech Debt

Will Larson's essay on running large-scale software migrations, anchored in his experience leading Uber's shift from Puppet-managed services to two-click self-service provisioning, argues that migrations are the only mechanism that actually moves the needle on technical debt at scale — and that there is a repeatable playbook for running them well.

---

## The argument in one paragraph

Larson claims that individual and team-level tech debt reduction is self-defeating in aggregate: every engineer grabs the easy projects, every manager schedules the isolated ones, so what remains is precisely the work that requires many teams coordinating — which is to say, migrations. Therefore an organisation's *migration capability*, not its code cleanliness, becomes the defining constraint on velocity as it grows, because most tools and processes survive only one order of magnitude of growth. This is falsifiable: if a company could meaningfully reduce debt through distributed local effort, or if its velocity were constrained primarily by something other than its stock of outdated platforms and patterns, the claim would fail. The essay's practical core is that migration execution is a learnable discipline with three phases — derisk, enable, finish — and that most migration failures are failures of sequencing and incentives, not technology.

## Key quotes

> "most tools and processes only support about one order of magnitude of growth before becoming ineffective, so rapid growth makes them a way of life."

The reframe that makes the whole essay work: migrations aren't a symptom of bad engineering. A tool failing at 10x is evidence it was *correctly* designed for the constraints it faced. This defuses the shame that usually surrounds replatforming.

> "If you don't get effective at software and system migrations, you'll end up languishing in technical debt. (And still have to do one later anyway, it's just that it'll probably be a full rewrite.)"

The parenthetical is the sharpest line in the piece. The alternative to a managed migration isn't avoiding the migration — it's an unmanaged one, on worse terms, at the worst possible time.

> "each team that endorses a migration is making a bet on you"

Larson locates the real currency of a migration in trust, not tooling. One abandoned migration makes every future one harder to staff, which is why he insists on starting with the *hardest* teams rather than the easiest — easy wins create false confidence that collapses at the first atypical codebase.

> "programmatically migrate the easy ninety-percent"

The inversion here is against instinct: most migration leads start by generating tickets. Larson says build the codemod first. The ninety percent that can be automated is where the org-wide cost lives, and driving it toward zero is what buys the political capital for the long tail.

> "it turns time into your friend. Instead of falling behind by default, you're now making progress by default."

The ratchet insight. Installing a linter rule that blocks new code on the old system converts a fixed-capacity migration team into a migration that advances with every commit, even ones made by people who've never heard of the project.

## Critical analysis

What's non-obvious here is the incentive analysis. Most migration writing is about tooling; Larson's most durable points are about *recognition structure* — celebrating kickoff rather than completion creates exactly the perverse incentive (start everything, finish nothing) that leaves organisations littered with half-migrated systems, which are worse than either state. His claim that "if a team isn't working on a migration, it's typically because their leadership has not prioritized it" is a quietly radical transfer of blame from engineering teams to the management layer, and it's correct: migrations are coordination problems, and coordination is what leadership actually controls.

The essay is also honest about the "running to stand still" failure mode — a team whose entire capacity is consumed by upgrades — without pretending there's a clean escape from it. That honesty is rare.

What's weak: the playbook is optimised for platform-type migrations (provisioning systems, build tools, infrastructure) where a codemod can touch the easy ninety percent. Product-code migrations with deep business logic embedded in the old pattern don't decompose as neatly, and the essay doesn't address them. The "finish it yourself" advice for the long tail is right but glosses over the case where the long tail is long *because* each item is genuinely hard, not merely unstaffed. And the piece predates the current agent era by years — it treats programmatic migration as something humans build by hand, which is exactly the assumption that agent-assisted migration is now breaking.

What it leaves out: any treatment of data migrations, where reversibility — his own stated property of good tooling — is often physically impossible; and any discussion of *when not to migrate*, the judgment call about which of the infinitely long queue of possible migrations deserves to exist at all.

## Related

- [[AI Code Migration with Claude Code]] — Anthropic's field report on agent-run migrations (Zig→Rust, Python→TypeScript) is the modern descendant of Larson's "programmatically migrate the easy ninety-percent": it strengthens his playbook by showing agents can now build the codemods he describes, while complicating his assumption that the tooling itself is the expensive, human-built part.
- [[Ratchets in Software Development]] — Larson's "stopping the bleeding" phase — installing a ratchet in your linters so new code must use the new approach — is a direct application of the ratchet pattern; this source gives the ratchet its most important use case, turning time into an ally during migrations.
- [[Platform Engineering End-to-End]] — Cavallin's platform-engineering guide treats migrations as one lifecycle stage among many; this source deepens that treatment, and its derisk-first sequencing (hardest teams first, trust as currency) nuances the platform guide's stakeholder-politics advice with a concrete mechanism.
- [[Patreon Notification Fanout]] — Patreon's notification-platform rebuild is a case study of exactly the kind of migration Larson theorises; this source's playbook (derisk with the hardest consumers, finish the long tail yourself) reads as a checklist for what Patreon's 200-type migration across 10 teams had to get right.

---
*Sources: [[raw/migrations]], [[summary/migrations]]*
