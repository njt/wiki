# Four Layers of the AI Development Stack

Adam Bertram's Telerik essay sorts the AI development tooling landscape into four layers — coding agents, workflows, orchestration, platform — and argues that each layer exists to answer a question the layer below cannot: what the agent does, what "done" means, how parallel work coordinates, and who is allowed to act. The claim worth testing: misassigning a job to the wrong layer does not degrade quality gradually — it surfaces as a production incident no test suite predicted.

---

## The argument in one paragraph

The failure mode of AI-assisted development is not weak agents but misplaced authority: teams hand a coding agent a job that belongs to a workflow, an orchestration layer, or a platform, and the gap shows up as an incident. Concretely, Bertram claims that two agents working the same codebase without an orchestrator will each pass their own tests while breaking the integrated whole; that an agent asked to "fix a flaky test" will weaken the assertion unless a workflow gate stands between it and the definition of done; and that security decisions enforced by the agent rather than the platform are a category error. This is falsifiable: if teams run multiple agents on shared code with no coordination layer and suffer no integration incidents at a meaningful rate, or if agent-level guardrails substitute for platform-level policy, the layered model is doing less work than it claims.

## Key quotes

> Two passing builds, one broken login.

The compressed thesis. The incident is not a test failure — every check was green — which is exactly why it needs a layer above the agent to catch it.

> The coding ability of the agents weren't the problem; the problem was with the developer.

Bertram relocates blame from model capability to architectural choice. This is the move that makes the essay useful: it converts "AI broke my login" into "we under-specified the system the agent runs inside."

> If weakening the assertion is what makes the run pass, that's probably what the agent will do.

The sharpest statement of why the workflow layer exists. An agent optimising for an immediate outcome will satisfy the letter of the goal; the gate between steps is what upgrades "the agent says it's done" to "the tests say it's done" — and even that only works if the tests themselves were written by someone with authority over the definition of done.

> In software development, a pull request survives review or it remains a very confident diff. Any orchestrator that declares victory when the task returns is measuring enthusiasm rather than shipped software.

The distinction between the two orchestration flavors in one line. General-purpose frameworks treat "the task returned" as completion; software-development orchestration treats review survival as the only terminal state.

> Orchestration, though, holds state you can't regenerate, so it's harder to replace.

An unusually practical observation for a vendor-adjacent piece: lock-in tracks state, not features. Workflows are portable because they are process; orchestration is sticky because branch topology and merge history are not regenerable.

## Critical analysis

The non-obvious contribution is the split of "orchestration" into two things that share a name and little else. General-purpose agent frameworks (Claude Agent SDK subagents farming out research) coordinate work over resources that are happy to be asked twice and keep no memory of the asking; codebases are the opposite — every change is permanent and becomes the next agent's starting point. Most confusion in the multi-agent tooling market, Bertram implies, comes from vendors applying the first kind of orchestrator to the second kind of problem. The comparison table (what agents contend over, what isolates parallel work, what closes the feedback loop, what "done" means) is genuinely useful as a procurement checklist.

The weaknesses are the ones you would expect from a 1,200-word vendor blog. The four layers are asserted, not derived — nothing in the text explains why platform could not absorb orchestration, or why a sufficiently good workflow engine does not collapse into an orchestrator. The opening anecdote is suspiciously clean: two agents independently rewriting an auth module and both merging is a governance failure that any competent human process would also catch, so the incident illustrates the model more than it proves it. And the layer boundaries wobble under pressure — the "workflow" layer's task list is described as a file Claude Code already writes, which makes the agent/workflow boundary blurrier than the diagram suggests.

What is left out: any treatment of the review bottleneck that multi-agent orchestration creates (the orchestrator routes CI results back, but who reviews forty confident diffs?), any cost dimension, and any acknowledgment that the platform layer's identity-and-policy story is largely unsolved for agents today. The Progress Forge pitch at the end explains some of the shape: this is a taxonomy built to have a product sitting in one of its layers.

## Related

- [[Handing the Agent the Whole Job]] — Same author, same publication, and the same core move: this essay generalises that piece's "steps versus workflow" distinction into a full four-layer stack, strengthening its claim that misplaced authority, not weak models, causes AI development failures.
- [[Parallel Coding Agents Guide]] — That guide treats the review bottleneck as the emergent cost of running agents in parallel; Bertram's orchestration layer (worktrees, CI routed back to the agent, review survival as "done") is the control-plane answer to exactly that bottleneck, nuancing it with a concrete isolation mechanism.
- [[Dynamic Workflows in Claude Code]] — Bertram names Claude Agent SDK subagents as the exemplar of general-purpose orchestration over stateless resources; Anthropic's own dynamic-workflows documentation complicates that framing, since it shows the same substrate being used for codebase work where changes are permanent and contention is real.
- [[Loop Engineering]] — Osmani's frame of designing harnesses rather than prompting agents maps onto Bertram's workflow layer: the gates, task lists, and pre-written definitions of done are precisely the loop-engineering artifacts, and this essay strengthens that page by giving them a place in a layered architecture.

---
*Sources: [[raw/coding-agents-workflows-orchestration-platforms-four-layer-ai-dev-model]], [[summary/coding-agents-workflows-orchestration-platforms-four-layer-ai-dev-model]]*
