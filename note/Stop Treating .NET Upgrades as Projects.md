# Stop Treating .NET Upgrades as Projects

A short polemic from dotneteers.net arguing that framework modernization fails when organized as projects — temporary budgets, big-bang migrations, rescue missions — and works only when built as a permanent engineering capability: continuously maintained compatibility inventories, characterization tests, upgrade recipes, and small reversible PRs. AI agents accelerate the mechanics; the real shift is organizational.

---

## Key quotes

> "A project has a beginning, an end and usually a large temporary budget."

The framing is the diagnosis. Projects reward a big push and then disband — exactly the wrong shape for a problem that recurs annually. Deferring upgrades makes each one *more* exceptional, which justifies a bigger project, which entrenches the pattern.

> "The longer they wait, the more exceptional the upgrade becomes. That is the real problem."

The escape isn't better projects; it's removing the conditions that make upgrades projects at all.

> "A healthy modernization capability continuously maintains: compatibility inventories, characterization tests, automated upgrade recipes, performance baselines, small reversible pull requests, explicit ownership of exceptions."

Six concrete artifacts of permanent hygiene. The list is notable for how mundane it is — characterization tests and baselines are boring, repeatable engineering, not heroics. "Explicit ownership of exceptions" is the honest part: some incompatibilities will exist, and someone must own them by name rather than letting them accumulate invisibly.

> "AI agents can accelerate much of this work. But the larger change is organizational."

A deliberately subordinate claim. Agents are the accelerant, not the strategy — the same move as [[Harness Engineering is not Enough]], where tooling alone can't carry the load.

> "The best upgrade project is the one you no longer need to create."

The abolition, not the optimization, of the migration program.

## Themes

#concept #pattern #tool

- **Capability over project** — recurring work modeled as continuous practice, echoing [[Ratchets in Software Development]] and [[Migrations — The Sole Scalable Fix to Tech Debt]].
- **AI as accelerant on boring work** — agents shine on characterization tests, recipes, and inventories; the organizational change remains human.
- **Small reversible PRs as risk control** — the same granularity discipline as incremental delivery everywhere.

## Analysis

This is a thin essay with a thick idea. It barely mentions .NET specifics — the annual release rhythm is just the concrete instance — and the argument generalizes to any platform or dependency upgrade. What's genuinely useful is the inversion: most orgs treat migration pain as evidence they need a *better* project next time; this says the pain is evidence of the project shape itself.

The AI angle is where it's most current and most understated. Agents change the economics of continuous upgrading: characterization tests, compatibility inventories, and mechanical API migrations are precisely the high-volume, well-specified, verifiable work agents do well. But the author is right not to lead with that — if you keep the project-shaped process and add agents, you just get a faster big-bang migration with more AI-written slop. The organizational move comes first; agents then make the continuous version cheap instead of merely principled. It pairs naturally with [[How to Prepare for AI-Driven Code Modernization Projects]] and [[AI Code Migration with Claude Code]], which cover the mechanics this piece deliberately leaves out.

Criticism: the six-part checklist is asserted, not argued — no evidence that continuous upgrading is actually cheaper at enterprise scale, where licensing, frozen subsystems, and vendor contracts genuinely do force batch migrations. The ownership-of-exceptions bullet quietly concedes this. Still, as a normative target it's correct, and the annual .NET cadence makes the old "wait for a stable LTS forever" posture untenable.

---

*Sources: [[raw/stop-treating-net-upgrades-as-projects]], [[summary/stop-treating-net-upgrades-as-projects]]*
*Last updated: 2026-10-08*
