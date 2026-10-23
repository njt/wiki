# Redesigning Source Control (Anthropic)

Anthropic's research write-up of Scion, the in-house version control system it built and migrated its monorepo to in five months, after agent-driven merges grew from ~2,500/day to 30,000+/day and git's offline-first, distributed design became the bottleneck. A case study in what "agent-scale infrastructure" means: always-online thin clients, server-side everything, sparse-by-design checkouts, and CI workloads measured in tens of millions of checkouts a day.

---

## Key quotes

> "The bottleneck, now, is all of the developer infrastructure built for human scale: source control, builds, and CI."

The clearest statement yet of the post-coding bottleneck thesis — and from the company with the most extreme data. Every downstream-infrastructure essay in this wiki (CI, review, merge queues) is a local instance of this one sentence.

> "We noticed that, like most companies, we were using git as a centralized system with one authoritative copy, but nearly all of our clients were agents and jobs that are online by definition."

The pivotal observation. Git's defining feature — offline, self-contained clones — is dead weight when the clients are agents that are *always* online. Scion is what you get if you design from that fact instead of retrofitting onto git.

> "Every commit, rebase, or merge happens on the server, so there's nothing to push... The cost of creating a commit... has to scale with the size of the change, not the overall size of the repo."

Changesets as write-ahead-log entries plus derived data computed recursively — this is a database design more than a VCS design. Tellingly, refs are discouraged: "global state that every server has to agree on is a looming scaling problem." Branches, the atom of git workflow, become a liability at agent scale.

> "Very often, we found that the role of the engineers was to tell Claude 'no.' Agents will sometimes violate core system constraints to satisfy a pattern that should never have been allowed in the first place."

The most transferable line in the piece. Examples are concrete: agents brute-forcing merge lookups by linearly scanning the commit log, and adversarial review driving over-engineering against hypothetical threats. Human judgment relocated to constraints and design tradeoffs, not lines of code.

> "A proper merge queue, for example, doesn't make much sense at agent scale. What does CI even mean when hundreds or even thousands of changes can merge while tests are running?"

Anthropic simply abandons the global repository lock and invests in post-merge/pre-deploy testing — betting that "Claude can typically fix these conflicts faster than humans can even identify them." That is a radical redefinition of CI, and it sits awkwardly with everyone still building tighter merge queues.

> "We created Slack channels where we tasked Claude with being obsessively focused on the health of a single component. We call these Claudes 'superfans.'"

Observability at agent scale includes agent observers. A superfan per component, with telemetry access, outperforming traditional alerts — ops as a fleet of agents, not dashboards.

## Key themes

#concept agent-scale infrastructure #tool version control #pattern server-side everything #pattern human-as-constraint-setter

## Analysis

The strongest thing here is the honesty about workload. Anthropic is candid that 25 million checkouts/day, 100 GB of history, and "latent demand for merging code [that] is practically infinite" are unusual — and that Scion "really only make[s] sense for truly enormous monorepos." This is not a git-killer pitch; it's a weather report from six to twelve months ahead, with the git community explicitly invited to adapt the ideas (server-side RPCs, better negotiation, composable sparse/shallow features) rather than migrate.

The strategic restraint is notable. They could have productized Scion; instead they argue that a bespoke everything-stack (VCS, review, search, CI) is "difficult for other organizations to adopt" and that building a public version "that tries to please everyone wouldn't actually be very useful." Whether that's wisdom or a moat defence is worth watching — Antique, the code review surface, gets one sentence and no detail, which is exactly the part the [[AI Code Review]] community would most want.

The human-role claims deserve scrutiny. "All of the engineers... would now rate Claude as better at *writing* code than them" is self-report from the most Claude-saturated company on earth, and the fifty-month, seven-person migration timeline is doing a lot of unexamined work — what did the migration *cost* in correctness risk, and how was that measured? The merge-queue abandonment is the piece's quiet confession: they couldn't replicate a global lock, so they redefined the problem as one they could solve. That's good engineering, but it's also the kind of trade-off that reads as principle after the fact. Still, "engineers tell Claude no" is the most precise articulation yet of the constraint-setter role the rest of this wiki keeps circling.

## Relation to the wiki

- [[Developer Tools]] — this is the first fully-fledged agent-scale version control system in the collection, giving real engineering substance to the "version control is overdue for a rethink" thread that [[DeltaDB]] and [[Twigg]] approach from the open-source side.
- [[Gas Town's Agent Patterns]] — Gas Town's agent-managed merge queues assume the merge queue as an organizing primitive; Scion's claim that merge queues "don't make much sense at agent scale" complicates that directly, and both documents agree the conflict-resolution burden shifts to agents.
- [[CI Speedup — Linear's CI Optimizations as Agent Prompts]] — Linear optimizes CI *within* git's model (merge queues, cache markers); Anthropic went underneath the model entirely. Same pressure point, opposite altitude.
- [[ctx – Agentic Development Environment]] — ctx's local merge queue is the quality gate for laptop-scale agent work; Scion shows what that gate becomes when the repo itself, not the workflow, is the scaling problem.
- [[RepoMirror]] — another Claude-driven overnight migration story, at hackathon scale; Scion is the same pattern (agent rewrote the ingestion pipeline 20x smaller, 4x faster) applied to production infrastructure with a five-month migration behind it.

---
*Sources: [[raw/redesigning-source-control]], [[summary/redesigning-source-control]]*
*Last updated: 2026-10-24*
