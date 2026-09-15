---
url: https://www.telerik.com/blogs/coding-agents-workflows-orchestration-platforms-four-layer-ai-dev-model
date_fetched: 2026-09-15
---

Summarize with AI:

Coding agents, workflows, orchestration and platforms do four different jobs, and swapping one for another surfaces in production as an incident no test suite predicted.

Two teams point two coding agents at the same authentication module. Both agents finish and pat themselves on the back. They each have achieved clean pull requests, green test suites and a deploy nobody lost sleep over.

Then production starts handing out 401 errors in the web app. But why? Each agent tested only its own branch. They each had no idea the other existed. If the branches were merged, the agents would have seen the two branches expose a token refresh path neither test suite had heard of. Two passing builds, one broken login.

The coding ability of the agents weren’t the problem; the problem was with the developer. Both teams handed their agent a job when they should have involved them in an AI workflow.

When it comes to developing software with AI, you can informally define the process as having four layers:

Each layer settles a question the one below it can’t answer. Skipping a layer leaves a gap that surfaces later as an incident.


The four layers of the AI development stack: coding agents at the base, then workflows, then orchestration, then platform at the top, with authority and scope flowing down the left rail and verified work flowing up the right rail.

Claude Code and Cursor’s agent take “add rate limiting to the ingest endpoint,” read the repository and rewrite the code until the test goes green. They will do that all night, with a worldview one instruction wide.

That worldview lasts exactly as long as the context window. But the definition of done and the state of what is in flight have to survive the session, in a place the next teammate can open and read. Teams still ask the agent to be that memory, which is a job it was never built to do.

A coding agent is tasked with a job. On its own, without guardrails, it can optimize for an immediate outcome even if that means it does the job wrong.

For instance, say you tell an AI agent, “Fix this unreliable rate-limit test.” That could mean that the agent’s goal becomes “make the test pass.” The agent edits the test until the run passes. If weakening the assertion is what makes the run pass, that’s probably what the agent will do. Now the test might turn green, but the underlying problem may not have been addressed.

Instead, moving the pass validation into a workflow adds a layer of accountability. Before an agent can declare success, it has to validate with the workflow.

The workflow gives that run a shape to follow instead of letting the agent improvise. The clearest example is the task list. Claude Code and Codex break a big job into verifiable subtasks and keep the list as a file committed to the repository a human can read.

The other half of the layer is the gate between steps: plan, execute, test, then stop until the checks agree. It upgrades “the agent says it’s done” to “the tests say it’s done,” the way a receipt turns “trust me” into a record. The definition of “done” lives in the workflow, written by the team before the agent starts.

*Pro Tip: A task list you can read mid-run is the difference between reviewing a workflow and interviewing a transcript.*

The moment two agents or two workflows are working in the same codebase, you need a system to coordinate them so they don’t conflict with each other. That system is orchestration, which ships in two flavors that share a name and little else.

**General-purpose frameworks** coordinate teams of specialist agents that are doing relatively independent tasks, like calling APIs or processing data. These agents don’t generally interfere with each other, so it doesn’t matter much if they are operating in the same space simultaneously. The Claude Agent SDK’s subagents are one example, farming research out to helpers. They were built for work where the shared resource is happy to be asked twice and keeps no memory of the asking. The resource is not changed by the agents accessing it. Codebases don’t forgive like that: every change is permanent and becomes the next agent’s starting point.

**Software-development orchestration** coordinates the moving parts a codebase runs on so it has to be much stricter. This type of orchestrator needs to keep agents’ work separate, track dependencies and validate code before declaring work finished. One common pattern gives each agent its own git worktree. The orchestrator tracks which branches exist and what each one depends on. It routes CI results back to the agent that made the change and follows every pull request through review to merge. It keeps the state of the feature in a store no agent owns, so the feature outlives the session that started it.

| Question | General-purpose orchestration | Software-development orchestration | 
|---|---|---|
| What do agents contend over? | APIs, records, queues | Branches, files, merge history | 
| What isolates parallel work? | Session or thread state | Git worktrees and branches | 
| What closes the feedback loop? | Retry or human review | CI results returned to the agent | 
| What does “done” mean? | The task returns | A pull request survives review | 

In software development, a pull request survives review or it remains a very confident diff. Any orchestrator that declares victory when the task returns is measuring enthusiasm rather than shipped software.

Governance is what happens between “the agent asked” and “the agent acted.” When we’re talking about production work, security and compliance controls should be enforced by the platform, not left up to the agent. The questions that follow:

An agent might request access to a production database, but the platform should vet that agent just like any other system or user.

When Claude Code asks to read the production schema, the platform checks which identity is asking and which policy applies before the query runs. Those controls are the ones you already run everywhere else, pointed now at the agent.

Every tool in the stack belongs to one of those four layers. It doesn’t matter what a vendor calls it, but what the tool actually does.

Ignore the label and ask what the tool makes reliable. “Plan, execute, test, fix” is a workflow, whatever the box says. “Durable state and mergeable parallel work” is orchestration. “Who can run what, where, with what audit trail” is a platform.

These layers also create different levels of vendor lock-in. Workflows are usually easier to replace. Orchestration, though, holds state you can’t regenerate, so it’s harder to replace.

The more of these questions you can answer yes to, the easier the tool will be to replace. If the answer is no to several of them, switching vendors later may mean rebuilding parts of the way your teams develop and ship software.

Progress Forge (formerly Progress Agent Harness) is a CLI-based orchestration layer for the AI coding agents developers already use, bringing structure, shared context, controls and traceability to AI-assisted software development. Book a Progress Forge Demo today!

Adam Bertram is a 25+ year IT veteran, former Microsoft MVP and self-employed consultant who helps organizations replace repetitive manual work with generative AI automation and agent-based workflows. He’s a successful blogger, consultant, trainer, published author and freelance writer for dozens of technology publications.
