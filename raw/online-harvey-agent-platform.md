---
url: https://ai.engineer/code/2026/talks/online-harvey-agent-platform
date_fetched: 2026-11-26
---

## Introduction

*This reading edition uses a descriptive title; the formal talk title was not verified.*

Joey Wang opens with the scale of Harvey's internal agent platform: more than one agent starting every minute on average, around the clock, and thousands of agent-written pull requests. The difficult part was not persuading a model to write code. It was keeping everything around those agents working as more people found ways to use them.

The platform, **Spectre**, had already escaped its original coding use case. Wang describes people asking data questions through Slack and a marketing colleague bringing a prototype and talk track to an agent to prepare material for a product video. Once a harness can reach the company's tools and run useful work, its users will bring tasks its builders did not anticipate.

That expansion is the point of the talk. A harness becomes shared infrastructure, and shared infrastructure has failure modes that a single successful agent run will never reveal.

## Own what happens after the code is written

At 01:39:24, Wang answers the obvious question: why build an internal platform when engineers already have favorite coding agents?

Harvey's answer is to run those agents, not replace them. The platform supplies company-specific secrets, a real test stack, alerts, tickets, shared policies, and capacity. An engineer can start with a preferred local tool and hand work into the company's environment.

The important workflow continues after the first patch. Wang describes an overnight alert becoming a proposed fix, a real stack being started to validate it, and proof being attached to the pull request. If continuous integration fails, a follow-up loop reads the failure and tries again. By morning, the change is waiting in the same review process as other changes, rather than stranded at the first error.

Human approval remains distinct from generating a patch or enabling automatic merging. Wang's recommendation is to build the follow-up loop if you can build only one piece. Its job is to carry a task through the ordinary friction between writing code and landing a reviewed change.

Projects give that work shared context. Teams record how services should be tested, pin trusted skills, and preserve an audit trail. The choice of coding agent can change without throwing away the team's operating knowledge.

## Make every action callable, then expect agents to call it

The second story starts with an interface rule: if a person can do something in the product, an agent should be able to invoke it programmatically.

Spectre exposed actions through a command-line interface. Starting a task, turning on a follow-up loop, and configuring later workflow steps no longer required a person sitting in the web app. That made the platform usable from laptop agents, agents already inside sandboxes, and CI.

*At 01:45:00, several different clients share the same execution layer. The interface is useful precisely because no single agent has to own the whole workflow.*

Wang describes coordinating a large task into milestones and dispatching many of them to Spectre, each with its own sandbox. He also describes a Dot collecting the questions Spectre is waiting on and drafting answers for his approval. The platform becomes a tool another agent can use, rather than a destination a human must continually visit.

This also changes what counts as a likely workload. A person clicking through a UI has a natural rate limit. A script or coordinator can create hundreds of tasks before anyone notices the cluster is full.

## Deduplicate the work, then make capacity fair

The first overload came from an alert storm. A release bug triggered an alert for each workspace; a workflow tagged the agent for every alert; every tag started a new task. Wang reports roughly nine hundred agents pursuing the same underlying bug over several hours.

The fix was to join a repeated alert to an investigation already in progress. More agents were not more independent evidence. They were duplicate demand.

The next overload came from a legitimate engineer assigning hundreds of tasks through the CLI. There was no alert storm to deduplicate. The platform needed admission control: accepting a request for work should not mean launching it immediately.

*The 1:46:50 slide distinguishes getting a place in the queue from getting a worker. Its performance numbers come from a synthetic 300-task load test, not a claim about every production request.*

The queue reserves room for interactive work, limits each person's concurrent tasks, allows borrowing when capacity is available, and shows users where their work is waiting. The objective is that one person, agent, or alert cannot take the whole cluster.

Wang is careful about attributing later improvements: several changes shipped together, so he cannot isolate which fix deserves the credit. The stronger evidence is operational. Bursts grew while the platform became better able to keep serving other work.

## Fix one bottleneck and meet the next one

Some latency had nothing to do with model reasoning. Repository checkout was expensive enough to justify keeping local copies on workers and lending them to tasks.

After task execution improved, the pressure moved downstream. Wang reports a sharp increase in company-wide pull requests and longer merge waits. A change must rebase, run CI, and remain compatible with a moving main branch. If main breaks, everyone inherits the failure. Generating more changes does not solve that coordination problem, and the talk explicitly leaves it unfinished.

Unattended automations add another source of demand. They need budgets, expiration dates, and a failure policy; Wang describes disabling an automation after five consecutive failures. A workflow that can wake itself must also know when to stop.

The closing questions concern proof and integration: can each proposed change demonstrate that it works before a reviewer sees it, and how does a team merge safely when agents never stop opening pull requests? Spectre's value lies in taking responsibility for those questions, regardless of which agent wrote the first patch.
