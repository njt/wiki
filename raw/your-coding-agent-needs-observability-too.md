---
url: https://www.mitchelsellers.com/blog/article/your-coding-agent-needs-observability-too
date_fetched: 2026-10-03
---

Most development teams would not intentionally run an important production workflow without logs, metrics, or traces. If you do, then you most likely haven't seen any of my talks, videos, or otherwise for the past 15 years, so be sure to check out some of my other content!

Yet that is effectively how many teams are adopting AI coding agents.

We can see that a developer started a session and eventually received a result. We may even have high-level usage numbers. But when a session takes an unexpected path, consumes more resources than expected, repeatedly calls the same tool, or produces a poor result, we have very little evidence to explain what happened. We just know that we know GitHub a certain amount of $$ for what we may have blindly used!

That is beginning to change. On September 22, 2026, GitHub announced OpenTelemetry support for the GitHub Copilot app, allowing enterprise-managed configuration to send agent activity to compatible monitoring tools. This appears to also flow down via enterprise settings into other tools as well, such as Visual Studio Code, etc.

The announcement matters, but the larger lesson matters more: as coding agents take on multi-step work, observability needs to become part of our AI adoption strategy.

## Adoption Metrics Are Not Execution Telemetry

Organizations naturally want to know whether people are using the AI tools they purchased. Adoption reports can help answer questions such as:

- How many developers are active?
- Which features are being used?
- How frequently are suggestions accepted?
- Where might additional training be useful?

Those are useful management questions. They do not explain an individual agent session.

An agent can call a model, inspect files, invoke tools, run a command, receive a failure, change direction, and call the model again. A usage dashboard may count that activity. A trace shows how the activity fits together.

That distinction is familiar from application monitoring. A monthly request count tells me whether an API is used. A distributed trace helps me understand why one request took eight seconds and where the time went.

AI coding workflows now need the same separation between aggregate reporting and execution-level troubleshooting.

## What OpenTelemetry Can Show

GitHub documents three categories of exported data:

- **Traces**connect the steps in an agent session, including model calls and tool usage.
- **Metrics**expose numeric patterns over time, such as input and output token usage.
- **Events**capture actions at a specific moment, such as whether a developer accepted or rejected an edit.

Together, these signals can help a team ask better questions:

- Did the session spend most of its time waiting on a model or executing a tool?
- Did an agent repeatedly read files without making progress?
- Are certain tools associated with more failures or rejected edits?
- Do particular workflows use far more tokens than expected?
- Did a change in configuration improve reliability or simply shift the problem?

This is not about watching individual developers. It is about understanding a new execution system well enough to improve it responsibly.

Over the past 18 months, we have seen GitHub change patterns, reports, displays, and methods to provide us with information. By logging this information on our own, we become empowered to report on the data in a way that works for us!

## Start With Content Capture Disabled

Observability data can create its own security and privacy risk, as we have talked about in prior blog posts about logging!

GitHub states that prompts, responses, and tool arguments are excluded by default. That is a sensible starting point because those fields may contain source code, filenames, internal architecture details, customer data, or secrets accidentally included in a prompt.

The managed-settings schema includes a `captureContent` option and a separate option to prevent users from changing it. A conservative starting configuration keeps content capture off and locks that decision while the organization evaluates the remaining telemetry.

This example is a starting shape, not a deployment-ready secret-management strategy. Do not casually commit a collector token to a repository. Decide how credentials will be delivered, rotated, and limited before enabling export.

Also decide who can query the resulting data. Removing prompt content reduces risk, but metadata can still reveal usernames, repository activity, tool choices, timing, and working patterns. Access control, retention, and audit expectations should be defined before collection begins.

## Collector or Directly to the OLTP Platform

GitHub supports exporting to an OpenTelemetry Protocol endpoint. You can point this directly to your OLTP platform, or you can utilize a Collector to validate and then ship the information off to the final destinations.

I utilize SEQ which has the ability to directly ingest, so I'm pointing things in directly right now, this may change in the future.

## Decide What Success Means Before Building Dashboards

It is easy to create a colorful dashboard that does not help anyone make a decision.

Start with a small number of operational questions. For example:

- Which agent workflows have the highest failure or rejection rate?
- Where do sessions spend most of their elapsed time?
- Are repeated tool calls indicating missing context or a weak instruction?
- Which tasks consume unusual token volume?
- Do policy or model changes alter success, latency, or cost?

Then build views that answer those questions. Avoid turning developer activity into a simplistic leaderboard. A developer working in an unfamiliar or highly regulated codebase may produce very different telemetry from someone doing repetitive work in a mature repository. Raw counts do not establish productivity or quality.

The most useful analysis will usually combine agent telemetry with delivery outcomes such as build failures, pull-request review time, escaped defects, rollback frequency, and developer feedback. Even then, correlation is not proof that the agent caused the outcome.

## Telemetry Does Not Replace Guardrails

Tracing an agent does not make the agent safe.

Organizations still need appropriate repository permissions, protected branches, review requirements, secret scanning, dependency controls, sandboxing, and limits on tool and network access. Content-exclusion policies can reduce which files become context, but they are not a substitute for those controls.

Observability helps us detect and investigate behavior. Guardrails limit what the system is allowed to do. Mature AI adoption needs both.

## Roll It Out as an Engineering Experiment

I wouldn't start by enabling every available field for every developer and keeping it on indefinitely.

A more practical rollout looks like this:

- Select a small, informed pilot group.
- Keep prompt, response, and tool-argument capture disabled.
- Document the purpose, access rules, and retention period.
- Validate the managed configuration and confirm clients actually receive it.
- Answer two or three specific operational questions.
- Review the results with developers before expanding collection.
- Adjust instructions, tools, policies, or training based on evidence.

This gives the organization a chance to learn what the telemetry reveals and what it does not reveal. It also creates room to discover privacy, volume, or interpretation problems before they affect the entire team.

## AI-Assisted Development Is Becoming an Observable System

AI coding agents are no longer limited to predicting the next few characters. They can perform longer workflows, use tools, modify files, and make decisions along the way.

That additional capability creates additional operational responsibility.

OpenTelemetry support is a useful step because it lets teams examine agent behavior using concepts and platforms they already understand. The goal should not be maximum surveillance or maximum data collection. The goal should be enough trustworthy evidence to troubleshoot failures, improve workflows, manage cost, and apply governance intelligently.

We would not accept an important production system that we could not observe. As agentic development becomes part of normal software delivery, we should apply the same expectation there.

Is your organization measuring only AI adoption, or can you also explain what happens inside an agent session when the result is slow, expensive, or wrong? I would be interested to hear which signals would be most useful to your team.
