---
url: https://tessl.io/blog/ai-devcon-nyc-from-coding-agents-to-software-factories
date_fetched: 2026-11-26
---

ARTICLE

# AI DevCon NYC: From Coding Agents to Software Factories

Patrick Debois

Curating a conference is an interesting way to see where an industry is heading, because the program tends to reveal which questions have already become accepted and which ones people are only starting to wrestle with.

For AI DevCon New York, happening November 2–4 at Industry City in Brooklyn, the central theme is **software factories**: not simply making individual coding agents more capable, but understanding the systems required to make agentic development repeatable, trustworthy and scalable.

The harder question now is how we move from one developer successfully using an agent to an engineering system in which agents can reliably participate in software delivery. Looking across the program, that system is starting to break down into a few recognisable layers.

## From coding agents to software factories

A working software factory increasingly needs:

- **Context and skills.**Agents need the specifications, rules and knowledge required to work effectively inside a codebase. Brandon Waselnuk's- *Emitted, not authored*looks at the context that never makes it into a README or skills file - the decisions and knowledge produced while teams are doing the work - and how agents can make use of it.
- **Harnesses and tools.**In London, Ryan Lopopolo coiner of the term harness engineering gave us a useful framing: the engineering around the probabilistic model that makes good behaviour more repeatable. Dru Knox, Head of Product at Tessl, will that into practice in New York by following a reliability loop from a failure to a guardrail and then into an improved harness.
- **Orchestration.**More agents do not simply mean more output. James Moss's session on what breaks when you go from one agent to ten looks at the practical coordination problems that emerge once agents share repositories, queues and workflows.
- **Verification and evaluation.**Faster generation moves more pressure into review. Nandini Singhal will show how Datadog combines telemetry, agent-session traces, task evaluations and delivery outcomes to understand whether agents are actually improving engineering work.
- **Feedback and improvement.**A factory needs to learn from what happened rather than repeat the same mistakes. Shivam Jindal's session on how SpaceXAI uses Cursor gives us one view of what that can look like in practice, with long-running agents working through investigation, review, CI failures and low-risk changes.

The interesting part is how these layers reinforce one another. Better context lets agents do more useful work, which increases the pressure on orchestration and review. Better verification permits more autonomy, which makes permissions and governance more important.

That changes the developer's role as well. Instead of correcting the same mistake every time an agent makes it, the scalable response is to understand why the system allowed the mistake and improve the harness, context or verification so it happens less often. At that point we are no longer simply using a coding assistant. We are operating a production system.

## The scaling challenge for enterprises

Things become harder again when we move from the individual developer to the organisation. One developer can install an agent, add local instructions and change their workflow overnight. A large engineering organisation has existing platforms, security requirements, legacy systems, compliance constraints and hundreds of teams that will otherwise solve the same problems independently.

This is why we wanted a strong enterprise dimension in New York. The useful questions are increasingly about how isolated successes become a shared capability:

- What belongs in the central platform, and what remains with individual teams?
- How should context and reusable skills be distributed?
- Which models and tools can access which systems?
- How do we understand security, cost and quality?
- How do we know whether any of this is actually improving software delivery?

Bartosz Ocytko's session on scaling agentic engineering at Zalando is one of the talks I am particularly interested in here because it covers the journey across more than 2,000 engineers, including platform investment, governance, risk-based review and knowledge sharing. That is the kind of experience that helps move the conversation beyond what works for a single developer.

At the developer level, people need to know how to work effectively with agents. At the team level, context and agent performance become shared concerns. At the platform level, organisations start needing harnesses, evaluation infrastructure, observability and safe execution. At the organisational level, somebody needs to know whether all of this is producing better software.

That broader practice is what I think of as **agent enablement**, and the software factory is the technical system that begins to emerge around it.

## Learning shoulder to shoulder

The other part of DevCon I care about is learning from people who are actively advancing the field by sharing their work while the practices are still forming. Geoffrey Huntley (creator of the Ralph Loop) and Steve Yegge (creator of Gas Town) are good examples of that: not because everyone should reproduce their approaches, but because seeing what people are trying helps expose the next set of problems earlier.

That same mentality is behind starting AI DevCon with a workshop day. A talk can explain the idea, but the details become much clearer when you try to build it yourself. Context turns out to be ambiguous, agents misuse tools in unexpected ways, and verification or orchestration problems that looked simple on a diagram become concrete very quickly.

That shoulder-to-shoulder mentality is important. The talks help move the ideas forward; the workshops and hallway conversations are where you compare them against your own experience and work out what you would actually change on Monday.

## What the program adds up to

What interests me most is that none of these areas exists independently. Better models create demand for better context. Better context increases agent throughput. More throughput creates pressure on orchestration and review. Better verification enables greater autonomy, which in turn creates new governance and platform problems.

That is what I mean by a software factory. It is not a single product, or simply another name for a coding agent. It is the collection of practices and infrastructure that begins to appear once software generation becomes one part of a larger automated development system.

AI DevCon NYC runs November 2–4 at Industry City in Brooklyn. My hope is that the useful part will not simply be seeing what people have built, but understanding where the current limits are and learning directly from the people working through them!

COPY & SHARE

Patrick Debois

Patrick is a pioneer, credited with coining the term DevOps, co-authoring the DevOps Handbook, and launching the very first DevOpsDays back in 2009. Since then, he’s been shaping the tech industry and brings depth across development, security, operations, and GenAI.

READING

·

0%

IN THIS POST

COPY & SHARE

Patrick Debois

Patrick is a pioneer, credited with coining the term DevOps, co-authoring the DevOps Handbook, and launching the very first DevOpsDays back in 2009. Since then, he’s been shaping the tech industry and brings depth across development, security, operations, and GenAI.

YOUR NEXT READ

## Your Software Factory: Dream Factory or Treadmill?

Your agents ship more code each week. Yet the same questions return in Slack, the migration never quite finishes, and every new engineer discovers the same gaps in the setup guide. The factory is b...

Patrick Debois
