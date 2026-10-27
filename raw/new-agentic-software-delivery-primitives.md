---
url: https://commandline.microsoft.com/new-agentic-software-delivery-primitives/
date_fetched: 2026-10-28
---

The promise of AI-first software development is extraordinary: describe what you want, and agents turn it into working software. But that framing treats the middle as a black box. Intent goes in, software comes out, and everything in between is assumed to simply be faster.

It isn’t that simple. Every stage of the software development lifecycle is changing, but not uniformly—some stages are transforming faster and earlier than others, and as capabilities improve, the bottlenecks move.

That’s why ambitious teams who try to transform the entire SDLC at once tend to struggle: when everything changes together, it’s hard to see what’s working, what’s failing, and what to fix. The problem is tractable, but only if we start with the primitives—the jobs to be done at each stage, the artifacts each stage produces, and the knowledge that connects them.

When we align on those primitives, the new software delivery system takes shape.

## The jobs remain while the work changes

Software engineering has always required us to define goals, create software, validate and deliver it, and understand what happens when people use it. Agents change where human attention is needed, but the underlying flow remains.

The SDLC has always followed a common workflow—several stages forming a loop: Plan, Build, Validate, Deploy, and Observe. In the last 20 years, we’ve introduced a wide variety of tools and automation to help us optimize how much time and energy we spend in each stage, but the stages themselves have stayed durable.

The new software delivery system retains these core stages with a key change: each stage now draws from (and contributes to) a shared foundation of knowledge. While the stages themselves are familiar, the human jobs to be done across the whole lifecycle change dramatically.

For each stage, we can make the transformation concrete by identifying the human job-to-be-done and the output artifact that carries the work forward. Trust is established at every stage in the lifecycle, and carries forward what it produced, the evidence behind it, and the questions that remain. That understanding then helps guide the decisions at the next stage.

| Stage | Purpose | Human JTBD | Artifact | 
|---|---|---|---|
| Plan | Define & measure goals | Choosing the right problems, resolving ambiguity, deciding on tradeoffs | Goals & acceptance criteria to guide creation & validation | 
| Build | Translate goals into a working system | Guide important design decisions + resolve problems that require human judgment | Candidate implementation (usually a PR) + context necessary to understand the choices made | 
| Verify | Get the system into real people’s hands | Judge whether the system has produced what the team set out to build | Auditable evidence of system behavior & a record of what was released, where, & under what conditions | 
| Observe | Operate the system & analyze the results | Interpret signals, develop insights, & decide what ships to production | Real outcomes that inform the next set of goals & system improvements | 

### Plan

Define and manage goals: what we want to achieve, the constraints we must respect, and how we’ll measure success. Human attention goes to choosing worthwhile problems and resolving ambiguity and tradeoffs. The output is a set of goals and acceptance criteria that can guide creation and validation.

### Build

Translate those goals into a working system. Agents take on more implementation work while humans guide consequential design decisions and resolve problems that require their judgment. The output is a candidate implementation (in the form of a PR), together with the context needed to understand its choices.

### Verify

Determine whether the implementation satisfies the intended goals, and when ready, put it in the hands of real users. Agents test and validate code across an array of dimensions (correctness, performance, security, etc.) and produce discrete, auditable evidence. Human energy shifts to the central question of “is the thing we built what we intended to build” and away from verifying implementation correctness.

### Observe

Operate the system, observe its behavior, and run experiments. Humans interpret signals, develop insights, and decide when and how users should experience changes. This stage produces signals and data which inform the next goals, experiments, and operational improvements.

## The knowledge at the center

Data and context form an essential part of the SDLC. Historically, much of that knowledge has been stored in the minds of individual developers rather than in shared records. At a startup, it may even be a single person! That person connects customer needs, design decisions, implementation, releases, and system operations. As the product and teams grow in complexity, the knowledge base fragments across more people and the ability to seamlessly connect data breaks down.

Within the new software delivery system, the knowledge layer becomes the new glue—it connects goals, design decisions, implementation context, validation evidence, release history, production signals, detailed agent behavior logs, and lessons synthesized from them. It keeps the source and version of each item, separates observations from interpretations, and serves the context each job is permitted to use.

So while the *idea* of knowledge is familiar, the ability to access and use that knowledge changes fundamentally. A production finding can shape validation, influence implementation, and change the next round of planning. Each stage uses and enriches the knowledge available to both agents and humans at any stage of the SDLC.

## From theory to practice

At Microsoft, we’re putting these primitives into action, building them into our developer products:

- Copilot can take an issue, spin up its own secure GitHub Actions environment, analyze the repo, change code, run validation, and produce a draft PR for human review. Developers can then delegate to the coding agent from GitHub, VS Code, mobile, and other surfaces.
- Copilot Code Review now supports agent skills and MCP, letting teams bring repo-specific standards and external context from issue trackers, docs, and other systems directly into reviews.
- And because the Copilot harness packages instructions, scripts, and supporting resources into reusable skills, successful ways of working become reusable production assets rather than staying trapped in an individual agent session.

While the agentic software delivery machinery matures and takes shape, we know that human involvement will continue to be essential now and in the future. Agents will help us achieve our goals, but it is up to us to identify the needs of users and to define, validate, and understand the products we create.

### Memory needs a purpose

The new central knowledge layer contains an incredible amount of context and data—signals from all different phases of the SDLC, user interaction, and agent behaviors. However, vast quantities of raw data do not improve agent behaviors. The data becomes useful when it enables agents to answer specific questions.

In the last year, we’ve started to describe this territory using words like “memory” and “dreaming.” Nomenclature aside, the point is bringing together data in a way that enables an agent to do its job better: avoid repeating mistakes, recognize relevant constraints, or improve decision-making based on past outcomes.

That gives us an engineering test and an evaluation. Does an agent perform a job better with the synthesized knowledge than without it? Is it more effective? Is it faster? Is it more efficient? Similar to a database index or a materialized view, the goal of memory is to give an agent quick access to data from the past to improve future performance.

### Tackle each stage of the SDLC independently, in parallel

Decomposing the new software delivery system gives us a practical way to make progress. For each stage, define the inputs, the jobs to be done by both humans and AI, the output artifact, and the criteria for success. Improve the agents with the same care used to validate the software they produce.

Those improvements must also add up to a better journey from idea to customer. Faster creation has limited value if validation can’t establish trust in the result. More experiments help when their findings improve subsequent decisions. The contracts and feedback between stages are part of what we need to engineer.

Even a loop that runs repeatedly without human intervention would still contain these responsibilities. It needs goals, an implementation, evidence that the implementation meets those goals, a way to reach users, and a way to learn from actual outcomes. Autonomy does not remove that structure.

Make no mistake: each stage of the lifecycle exists within the new software delivery system—even if we stop thinking about it explicitly as trust builds. But for now, we *must* think in terms of durable components as we stand up our initial software delivery systems in order to ground the human jobs to be done.

## More room for human judgment

There are important human jobs at every stage of today’s SDLC. As this system matures, our attention will increasingly concentrate on the places that require higher-level judgement. In the near term, I suspect we’ll be most concerned with validation, operation, and planning: determining if the code agents produce is trustworthy, improving how our systems run in production, and rethinking how we define goals for the software delivery system.

These jobs require judgment about value, intent, and customer experience. A software delivery system can execute a loop autonomously without establishing that its goal was worth pursuing in the first place. Humans and a refined sense of taste are necessary to identify and produce lasting value.

Building dependable software delivery system primitives gives us a way to expand automation while continuing to improve the quality of our decisions. As agents take on more of the work, we gain more time to define ambitious goals and bring incredible new experiences to life.
