# The Explorer Pattern (Larson)

Will Larson (CTO at Imprint) names the "Explorer Pattern": one person or a tiny team builds a working implementation of an ambiguous product problem while minimizing cross-team impact, then reconciles with other teams afterwards — accepting that the initial build may need substantial rework. The essay is less about a trick and more about a claim that 2026's economics broke the process habits built to avoid rework.

---

## What it argues

Before 2026, code was expensive and process was designed to minimize implementation and rework. Now generation is "effectively-free across many dimensions" — code, designs, analysis — and, more importantly, **reworking software is extremely cheap**. Larson says industry always optimized around the wrong constraint: "bringing engineering into problem-solving late 'to protect their time'" was designed as if writing code were the bottleneck, which "was never true." Cheap rework flips the optimal process: explore by building, not by aligning.

The organizing metaphor is database replication. The explorer is the **primary**: all work flows through one small group, so it is internally consistent and verified upstream. Other teams are **replicas**: they may be delayed but can't be wrong. He contrasts this with Dynamo/Cassandra-style quorum designs — reliable, tolerant of node loss, but slower and prone to inconsistency — which is what heavy upfront cross-team coordination amounts to. Consistency at the primary is necessary but not sufficient: "you can always insert data that meets the table's requirements but is still inherently wrong" (a valid `200` isn't the right credit limit).

## Key quotes

> "**Reworking software is extremely cheap**. Historically industry has optimized around avoiding engineering rework because engineering was the highest cost / narrowest constraint."

The load-bearing claim, and the one that generalizes the pattern beyond Imprint: if rework is nearly free, almost every process whose purpose was avoiding it is now mispriced. That's a sharper version of the "generation is cheap" argument most essays stop at.

> "The only way to navigate complex ambiguous problems is to hold things constant, even when you know they are imperfect. … just anchoring on a plausible guess to unblock us is much more valuable than trying to get them accurate. A little prework is useful, but a lot is just artificial precision that's a distraction from progress"

The headcount-planning example (revenue depends on forecast depends on partner go-live dates nobody knows) is the essay's most transferable move: pin a guess, move, revisit.

> "once a project goes beyond ~3 weeks of effort, if you can't very explicitly identify who the stakeholder is sponsoring your project and they would not enthusiastically agree, then you are likely closer to 'lost' than 'exploring'"

A crisp falsifiable line between exploration and drifting, and the rule most organizations will fail: sponsor accountability plus a five-person ceiling plus mandatory public progress reporting is what keeps "exploration" from becoming a euphemism for unowned work.

> "features and products move fast when we get the pieces working every step from frontend to backend to data pulls. This is the first step of exploration. Just make sure it's behind a feature flag that's only enabled for internal folks!"

Ship-to-production-behind-a-flag as the *first* step rather than the last: the whole pipeline gets exercised while the thing is still a hypothesis.

## Critical take

The strongest part is the naming itself. "Tech-led development" was a slogan; "primary-and-replicas" is an operational model with a failure mode attached — replica lag — which Larson treats as a managed quantity ("cut the replica-lag across teams down to near-zero" before shipping the Plaid feature). That gives managers something to reason about: how long can replicas stay behind before the reconciliation cost exceeds what upfront alignment would have cost?

The honest caveat is the one the essay states and doesn't fully price: "this approach can require very substantial rework to that initial development." Cheap rework is cheap for the explorer, but the reconciliation tax lands on *other* teams, and "make the narrowest viable changes to what already exists" is doing enormous work in that sentence — most teams that try this will not have the discipline, and will invalidate others' mental models at scale. It also presupposes an environment like Imprint's: small, high-trust, already agent-saturated. The pattern's rules (sponsor, five-person cap, public channel, feature flags) are guardrails against its own abuse, which is a good sign, but "hold space for being totally wrong" plus no roadmap is a governance posture only a founder-adjacent CTO can write.

## Relation to the wiki

- [[Trying the Software Factory Pattern (Larson)]] — same author and org, and the two are complementary halves of Imprint's 2026 operating model: the factory loop drives *known* projects; the Explorer Pattern handles the ambiguous ones that never made it onto a roadmap. The Plaid example even ships "never officially on a roadmap," which the factory's Linear-loop machinery presupposes.
- [[Rewrite All the Code All the Time]] — that piece argues wholesale rewrites are now rational because generation is cheap; the Explorer Pattern supplies the *organizational* wrapper that makes deliberate rework socially survivable, i.e. how you get a team to accept throwing the first pass away.
- [[The New Software Lifecycle]] — both describe lifecycle changes driven by cheap generation, but Larson's is the more concrete org-level protocol: where the lifecycle pieces describe phases, the Explorer Pattern prescribes who works, how large a group, and what gets published when.

---
*Sources: [[raw/explorer-pattern]], [[summary/explorer-pattern]]*
*Last updated: 2026-11-26*
