# Zero Alignment

Maggie Appleton's argument (from a GitHub research presentation) that the real bottleneck in AI-assisted development isn't implementation -- it's team alignment. One developer with two dozen agents and zero alignment produces chaos, not software. "Agreeing on what to build is the new bottleneck."

---

## Key Quotes

> "Software is not made by one person in a vacuum. It's a team sport."

> "The hard question is no longer how to build it. It's should we build it."

> "When production is cheap, opportunity cost becomes the real cost."

> "All these tools are single player interfaces."

## Key Themes

#orchestration #management #agentic-coding #discovery-vs-delivery

The critique is sharp: existing tools (GitHub, Slack, Jira, Linear) are single-player interfaces not designed for agentic workflows. Planning happens in fragmented places, implementation accelerates dramatically, review happens too late when changes are locked in. The pull request bears impossible weight as the only coordination mechanism.

This creates waste: merge conflicts, duplicated work, massive review backlogs, and effort spent building the wrong thing fast. The faster agents work, the more expensive alignment failures become.

GitHub Next's prototype **Ace** points toward the solution: multiplayer sessions backed by cloud microVMs where teams discuss features, prompt agents collaboratively, see shared previews, and maintain continuous alignment. The key shift is from asynchronous coordination (PRs, issues, Slack threads) to synchronous collaborative development.

The deeper argument: reclaimed implementation time should fund deeper thinking, better research, and higher craft. Quality becomes the differentiator in a world of cheap code generation. This connects directly to [[When the Target Keeps Moving]] -- if delivery is cheap, invest in discovery. And to [[Radical Accountability]] -- if building is easy, bad software reflects bad judgment, not insufficient resources.

## Critical Analysis

Appleton correctly identifies the gap: all the agent tooling optimizes for individual developer productivity, but software development is a team activity. The coordination problem isn't solved by making each individual faster; it's often made worse.

The weakness: Ace is a prototype from GitHub Next, not a shipping product. The vision is compelling but the execution path is unclear. Synchronous collaborative development works for collocated teams; distributed teams (most of the industry) need something that works asynchronously too. Still, naming the problem -- zero alignment as the failure mode -- is the necessary first step.

The structural answer to "should we build it" is the hierarchy [[Backlog Hierarchy Problem]] describes: separate objectives from initiatives from ideas so the "what matters" conversation happens at one altitude instead of being relitigated per item, and keep a two-backlog split (opportunity vs delivery) so planning doesn't have to share a column with execution. Its critique of Jira epics as "buckets, not strategic constructs" is the same single-player-interface gap Appleton identifies, stated at the backlog level.

---
*Sources: [[summary/zero-alignment]]*
*Last updated: 2026-05-14*