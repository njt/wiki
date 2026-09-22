# Shaped by Demand — The Power of Fluid Teams

Dan Terhorst-North's Craft 2025 talk argues that the demand is fixed and the teams are the variable: instead of slicing work to fit stable, long-lived feature teams, make all the demand visible, allocate discretionary spend, and let 60–200 people self-assemble quarterly under explicit constraints. Along the way he buries Tuckman's forming-storming orthodoxy on its own bad methodology, elevates failure demand to the silent capacity killer it is, and decouples leadership ("the person who cares enough to pitch") from management ("manages a system of work that has people in it").

---

## What the talk argues

- **The eight-person problem was solved; alignment at scale wasn't.** Cross-functional teams fixed the 1990s mess where the analyst, designer, programmer and tester sat in eight different teams. How 80, then 800 people point in the same direction is "the interesting question of the last quarter century" — and consultancies answering it with cookie-cutter approaches are optimizing "billable hours."
- **"The work" is five things.** Delivery, discovery (problem- and solution-space), kaizen (improving the whole system of work — teaching someone counts as first-class work), BAU, and failure demand. Only the first three are discretionary spend; backlogs show only delivery, so the mandatory two-thirds hides in plain sight.
- **Failure demand is the capacity killer.** Gene Kim's HP LaserJet firmware case: ~95% failure demand, two hours a week of discretionary time, eight months of dull automation to get to 40%. Root cause: "they didn't know how bad their failure demand was because they weren't looking at all the demand."
- **Stable long-lived teams rest on a myth.** Tuckman's meta-study was 50% therapy groups, 25% HR training, 25% actual work groups — "none of it's true." Later studies show the stages run concurrently and fast; with shared culture and a high hiring bar (ThoughtWorks), teams perform "within days, sometimes hours." Long-lived teams also presume homogeneous work, which Bank of America's 30–40 legacy systems refute quarter on quarter.
- **The inversion.** Demand is "the fixed piece." Demand-led planning: all demand visible, discretionary spend allocated with a no-zero rule, sponsors pitch to one room, people self-assemble under governing constraints (3–10 people; full autonomy = never blocked on another team; all pitched work done) and enabling constraints (mover/tenure dials). "Bumblebees" rove (max three teams each, three per team). Identity lives in the program, not the squad — Spotify got tribes right and then froze squads into the stable unit.
- **It compounds.** John Lewis went from two exhausting days with literal tears to planning 100 people's quarter by lunchtime: "it starts to be deltas after a while."

## Key quotes

> "The goal of organization design is to build the best structure we can to do the best work we can for now, knowing that it's going to change."

The definition is pithy, but the load-bearing word is "for now" — any structure is a quarterly hypothesis, not an org chart to defend.

> "Failure demand is work you should never have had to do."

Outages, reconciliation failures, "the meetings and the meetings about the meetings." The cheapest capacity you will ever find is the capacity you stop wasting.

> "Guess what, kids, it doesn't matter what methodology you're following, if you've got two hours a week to get work done."

The HP LaserJet team at 95% failure demand. This is the talk's sharpest anti-methodology point: process debates are irrelevant below a discretionary-spend floor.

> "It turns out the only problem with that story is none of it's true."

On forming-storming-norming-performing, right after revealing 75% of Tuckman's groups weren't work groups. The demolition matters because stable teams are the load-bearing assumption of every scaled-agile certification.

> "The point isn't the plan. The plan is going to change. The point is getting everyone in the room."

His answer to the program manager who rage-quit over 90 people spending two days on a plan one person could draft. The room is a shared-context machine, not a planning inefficiency.

> "Autonomy without that alignment is anarchy... alignment without autonomy is autocracy."

The two failure corners; the target is "the minimum amount of guidance, the minimum constraints, such that people can do good work" — then "they will surprise you." Explicitly mission-command lineage.

> "The team is the whole group... you happen to be with this configuration of half a dozen jokers for the next quarter."

Identity belongs to the program; squad membership is "kind of an implementation detail." This is the piece the Spotify model got backwards once squads went long-lived.

## Key themes

- **#concept** — Failure demand and the five-work taxonomy: discretionary (delivery, discovery, kaizen) vs mandatory (BAU, failure demand) spend, and measuring what share of capacity goes to work that shouldn't exist.
- **#pattern** — Demand-led planning: quarterly whole-room self-assembly under governing + enabling constraints, bumblebee rovers, BHAGs as aspirational OKRs, forward-only reviews ("I'm not interested in what you've done. I'm interested in how much is left").
- **#concept** — Autonomy through alignment: rules of the road as the mechanism, minimum constraints as the goal, anarchy and autocracy as the corners to avoid.
- **#person** — Dan Terhorst-North: BDD originator, ex-ThoughtWorks, working org design for 10–15 years; de-jargones on purpose — "I use grown up words like risk and governance now. It's wicked."

## Opinionated take

The genuine contribution is the inversion, not the playbook. Once you accept demand as the fixed piece, org design stops being a topology problem and becomes a scheduling problem — and most scaled-agile furniture (stable teams, fixed cadences, permanent squad membership) becomes an implementation detail. That reframing survives contact with the talk's own weak evidence better than any individual practice does.

Because the evidence is the weak point. He shreds Tuckman for a meta-study of therapy groups, then answers with anecdotes and one sentiment survey — he half-concedes it himself ("empirical, anecdotal sense... before Dave Snowden comes for me with pitchforks"). "I feel connected to the work I'm doing" at 4s and 5s measures how the room feels, not whether fluid teams ship more or better. A fair version of this talk would trade the Tuckman standard it applies to others for its own claims: throughput, quality, retention, fluid vs stable, same org.

The unspoken prerequisite is trust. The model demonstrably ran at ThoughtWorks (high hiring bar, shared culture) and John Lewis (two brilliant facilitators, years of runway, suits pre-briefed until they were comfortable "with me taking 100 people in a bag and shaking them up"). In a low-trust organisation the same room produces politics, not self-assembly — "tears and tantrums" is doing load-bearing work in that sentence. And the gap list is long and real: no answer for 500 people ("It's not supposed to [work]"), for the sad unstaffed reporting project, for annual budgets funding quarterly plans, for remote ("It's a bit rubbish, not going to lie"), for the middle-management layer being disposed of, or for the person who needs stability and gets a max-tenure constraint instead.

The quiet best idea is the on-call→kaizen loop from the Q&A: on-call signals become kaizen pitches next quarter, converting mandatory failure demand into discretionary improvement work. That flywheel is what pays for the whole scheme, and it's the piece most likely to survive translation into an agent-era org — where failure demand (rework, hallucination cleanup, review queues) is exactly the invisible capacity killer no backlog shows.

## Related pages

- [[Architecture Is Designing Knowledge Flow]] — the sibling Craft 2025 talk: Montalion argues transformation means changing the mindset, not the topology, while North supplies a concrete quarterly mechanism for changing the structure; both land on "the point is getting everyone in the room" as the actual deliverable.
- [[Who Does What — Team Topologies for the Agentic Platform]] — Team Topologies, like Spotify squads, treats team types as durable; North complicates that by making composition churn under enabling constraints and moving identity up to the program level.
- [[Building World-Class Engineering Teams in the Age of AI]] — Rajan's "don't be a manager" and collapsing layers are the same decoupling North makes explicit: lead by pitching the work, managers manage systems of work, appraisals move out of band to guilds.
- [[Martin Fowler and Kent Beck on Reinventing Software]] — North's jabs at the "big A agile industrial complex" selling certifications read as a practitioner's coda to the Agile founders' own ambivalence about what their movement became.

---
*Sources: [[raw/shaped-by-demand-the-power-of-fluid-teams-dan-terhorst-north-craft-2025]], [[summary/shaped-by-demand-the-power-of-fluid-teams-dan-terhorst-north-craft-2025]]*
*Last updated: 2026-09-13*
