---
url: https://www.telerik.com/blogs/multi-agent-orchestration-software-delivery-patterns-multi-repository-workflows
date_fetched: 2026-09-29
---

Summarize with AI:

Learn how to orchestrate software changes across multiple agents and repositories while managing handoffs, dependencies, context and delivery.

Your platform team hands a slew of authentication-service changes to AI agents. Ninety minutes later, 11 pull requests land across five repositories. Every test suite is green. But somehow the authentication service is still broken.

The agents did their jobs. Each “fixed” the issue it was assigned, but no one owned the part between them. None saw that Agent A’s change touched code Agent B depended on.

An agent, like a human, knows only the context it is given; without context it flies blind.

There are two coordination problems here. Multi-**agent** orchestration coordinates multiple agents working on separate tasks. Multi-**repository** orchestration coordinates changes that span codebases with different interfaces, dependencies and delivery gates. They often overlap, but solving one does not automatically solve the other.

The first problem is coordinating the agents themselves. That is the job of the multi-agent orchestration layer, a control layer above the coding agents. One framework you can use splits that control layer into three roles:

As those tasks cross repository boundaries, orchestration also has to account for the second problem: each repository may have different dependencies, interfaces and delivery controls.

Your CI/CD system holds many of those controls orchestration must enforce. Branch protection, code owners, semantic versioning and migration ordering were designed for humans working in parallel. Orchestration should make agents inherit those controls rather than create separate ones.

Inheriting those controls is harder than it sounds, because no two repositories enforce them the same way. Every repository has a gate, the check a change must pass before it merges. One repository might be a monorepo, a single repository that holds many projects and whose gate is a merge queue. Another might live in Azure DevOps, its gate approval from a named reviewer group. A third repository’s gate might be a nightly build and a person who says yes.

Each kind of gate needs its own integration with the CI/CD system that enforces it, tooling built to move a change through that gate. No single integration can handle all three, so the plan must record which repository’s gate enforces each requirement.

Get that inheritance wrong and it shows up in delivery numbers. DORA’s 2025 research still associates AI adoption with rising delivery instability.

Multiple agents are most useful when they can work at the same time. But parallel work is only safe when each agent is isolated in its own workspace.

If three agents are working from the same checkout, they can overwrite each other’s changes, create conflicts or leave the codebase in an unpredictable state. That is why isolation should come before parallelism.

One way to handle this is with Git worktrees. A worktree gives each agent its own working directory, index and HEAD, while still sharing the same underlying repository. Each agent gets its own checkout, so changes in one workspace do not interfere with another.

An agent told to stay in its tree can still wander into another. Only OS permissions or sandboxing draw a line an agent instruction cannot cross.

Two rules can help make isolation more reliable. First, a known base. When every worktree starts from a fresh `origin/main`, residual state won’t leak in from a previous run. Second, a single agent can edit a hotspot file at a time. Lockfiles and dependency injection registries are the usual hotspots, attracting edits from many different tasks, which can mean unnecessary conflicts and cleanup.

Even if each AI agent works in its own separate workspace, the agents can still fail when they pass work from one to another.

The coordinator hands a task to an implementor, who hands the result to the verifier. Each of those transfers is a handoff. The problem is that important information can get lost during those handoffs. Requirements, constraints, assumptions or context may not be passed along clearly. This workflow context needs to outlive an individual agent session.

Isolation can help protect the codebase from conflicting changes, but it does not guarantee that the right information moves with the work. Agents can still misunderstand one another, omit important constraints or make assumptions the next agent does not share. Research on multi-agent system failures has found that many failures come from this kind of misalignment and from the way the workflow is designed, not necessarily from the capabilities of the model.

Design failures have design fixes. Treat the handoff as a versioned artifact that can be reviewed like the code it ships with. A useful handoff should include four fields:

| What the handoff carries | Why the next agent needs it | 
|---|---|
| Scope | Defines which paths/files this task may modify, so the verifier can reject an out-of-scope edit | 
| Interface requirement | States the contract the work the task must satisfy, such as the method signature or the schema version other repos compile against | 
| Base | Records the commit the worktree started from, so a stale plan is detectable instead of silently wrong | 
| Evidence | Lists the commands or tests the implementor ran and their results, so verification starts from concrete fact | 

A strong handoff gives the next agent enough context to understand what changed, what constraints still apply and how the work has already been checked.

But those four fields often require more context than the coordinator can see inside one repository.

A dependency that matters usually sits in another repository, the shared SDK your services compile against or the schema your event consumers parse. Seeing past one repository is the difference between scoping a task and guessing.

If context only exists in an agent’s session, it disappears when the session ends. Plans, decisions and run history should be written down in a durable place, such as a dedicated Git branch, where they can be reviewed later and tied back to the commits they influenced.

Tools should be durable too. An integration written once against the Model Context Protocol interface belongs to your architecture, not one agent’s configuration. What comes next is composition—bringing all that isolated work back together into a single, coherent change.

Eleven pull requests can all pass their individual checks and still fail when they come together.

That is especially true when a change spans multiple repositories. One update may depend on another landing first. An infrastructure change might need to happen before a migration can access a secret, and the migration may need to finish before an application ships code that depends on the new data or claim.

Those dependencies create a required order of operations. Composition has an order. Per-repository CI does not see this order by default. Multi-repository pipelines exist in GitHub Actions and Azure Pipelines, but they have to be designed deliberately.

The 11 branches have to exist together as one state. The integration run, one CI run against that combined state, is where the interface requirements get re-checked, which per-branch checks can’t see. Agents can assemble all of it, but the decision to merge stays with the human reviewer.

Larger context windows do not remove the need for this coordination. Giving one agent access to every repository may reduce handoffs, but it also puts the entire change in one worker. If one part goes wrong, it becomes harder to isolate or reject just that piece.

Even with good coordination in place, the next bottleneck is often human review. Ten agents feeding one reviewer is a queue. Orchestration can route and summarize that queue, but it can’t replace the judgment required to work through it.

Take a cross-repository change your team shipped last quarter and rebuild the plan it would have needed, down to the files each task owns and where it goes stale as main moves on. Run it read-only, agents proposing and nothing merging. The gaps will surface in the first hour, almost never in the code the agents write. They’re in the interface requirements that weren’t written down.

Write them down. That is the prerequisite: a delivery workflow explicit enough to hand to something that will not ask you what you meant.

If you’re ready to move from isolated AI coding tasks to repeatable engineering workflows, Progress Forge (formerly Progress Agent Harness) is designed for exactly that shift. It orchestrates the AI coding agents your team already uses through structured workflows with visibility, governance and human review built in.

Explore the Progress Forge Early Access Program to see how it can help you put these ideas into practice.

Adam Bertram is a 25+ year IT veteran, former Microsoft MVP and self-employed consultant who helps organizations replace repetitive manual work with generative AI automation and agent-based workflows. He’s a successful blogger, consultant, trainer, published author and freelance writer for dozens of technology publications.
