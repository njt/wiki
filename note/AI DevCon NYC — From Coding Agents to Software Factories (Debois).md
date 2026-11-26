# AI DevCon NYC — From Coding Agents to Software Factories (Debois)

Patrick Debois — the person who coined DevOps — curates the AI DevCon NYC programme as a field survey of where agentic engineering is heading, and uses it to argue that the industry has moved past the question of individual agent capability into the question of the *system*: the five-layer "software factory" (context, harness, orchestration, verification, feedback) and the organisational practice he calls **agent enablement**.

---

## What it argues

The framing device is a programme preview: a curated conference reveals which questions have become accepted and which people are only starting to wrestle with. Debois reads the NYC programme (Nov 2–4, 2026, Industry City, Brooklyn) as showing the central theme is now *software factories* — "not simply making individual coding agents more capable, but understanding the systems required to make agentic development repeatable, trustworthy and scalable."

He breaks a working factory into five layers, each anchored to a talk:

- **Context and skills** — Brandon Waselnuk's *Emitted, not authored*: the decisions and knowledge produced while doing the work that never make it into a README or skills file.
- **Harnesses and tools** — Ryan Lopopolo (coiner of "harness engineering"): the engineering around a probabilistic model that makes good behaviour repeatable; Dru Knox follows a failure → guardrail → improved-harness loop.
- **Orchestration** — James Moss on what breaks going from one agent to ten, once agents share repositories, queues and workflows.
- **Verification and evaluation** — Nandini Singhal on Datadog combining telemetry, agent-session traces, task evals and delivery outcomes to ask whether agents actually improve engineering.
- **Feedback and improvement** — Shivam Jindal on how SpaceXAI uses Cursor: long-running agents working through investigation, review, CI failures and low-risk changes.

## Key quotes

> "Instead of correcting the same mistake every time an agent makes it, the scalable response is to understand why the system allowed the mistake and improve the harness, context or verification so it happens less often. At that point we are no longer simply using a coding assistant. We are operating a production system."

This is the DevOps move restated for agents — blameless postmortem logic applied to agent failures. It's the cleanest one-sentence definition of what distinguishes factory thinking from tool usage.

> "The interesting part is how these layers reinforce one another. Better context lets agents do more useful work, which increases the pressure on orchestration and review. Better verification permits more autonomy, which makes permissions and governance more important."

The compounding claim is the essay's real thesis: each layer's improvement *raises the stakes on the next*. That's why partial adoption plateaus — teams that buy context without verification simply generate more review debt.

> "One developer can install an agent, add local instructions and change their workflow overnight. A large engineering organisation has existing platforms, security requirements, legacy systems, compliance constraints and hundreds of teams that will otherwise solve the same problems independently."

The enterprise section is the least novel but the most operationally useful part: the questions (central platform vs. team autonomy, skill distribution, model access, cost/quality observability) are the standard platform-engineering questions re-asked for agents.

> "That broader practice is what I think of as **agent enablement**, and the software factory is the technical system that begins to emerge around it."

A deliberate parallel to DevOps-style enablement roles — the claim that a new job function, not just new tooling, is what makes scaling work.

## Key themes

#concept — the five-layer software factory (context / harness / orchestration / verification / feedback), each layer compounding pressure on the next
#concept — "agent enablement" as the organisational practice wrapping the factory
#person — Patrick Debois reading a conference programme the way he once read deployment pain: as a systems problem, not a tooling problem
#pattern — fix the system, not the instance: repeated agent failures are harness/context/verification bugs

## Opinionated take

This is a curator's essay, not a deep one — but its value is exactly that. Debois is doing to agentic engineering what he did to deployment: naming the stack so people stop arguing past each other. The five layers map almost one-to-one onto the conversations already happening elsewhere in this wiki, and the compounding claim ("each layer's improvement pressures the next") is the best short argument for why piecemeal agent adoption stalls.

The weak point is the enterprise section, which stays at the level of questions. "How should context and reusable skills be distributed?" is a good question with almost no distribution answers in the wild yet — the Zalando talk he flags (2,000+ engineers, risk-based review) is one of the few sources with data at that scale. There's also a mild conflict of interest worth noting: Tessl, whose blog this is, is selling exactly the factory scaffolding being described, and Debois's employer-adjacent framing ("agent enablement" as a discipline) conveniently implies demand for platforms like Tessl's. The layering is still honest; the motivation isn't neutral.

## How it relates

This source strengthens [[Software Factories, Light and Dark]] by giving the factory thesis a practitioner-conference census — the same stack (harness, verification, back pressure) appears as the actual programme of a major event, not one writer's model. It nuances [[Two Jobs — Product Engineering and Factory Engineering (Lloyd)]] by supplying the layer decomposition and the "agent enablement" job function that Lloyd's factory-engineering role implies but doesn't name. It complicates [[Building an AI-Native Engineering Org (Gondala)]]: Gondala's single-org field report sits at exactly the developer→team→platform ladder Debois describes, but Debois adds the 2,000-engineer Zalando data point as the missing large-scale end of that ladder. And it cross-checks [[Agentic Verification]] — Datadog's telemetry-plus-evals session is the observability-heavy version of the verification bottleneck Osmani names.

---
*Sources: [[raw/ai-devcon-nyc-from-coding-agents-to-software-factories]], [[summary/ai-devcon-nyc-from-coding-agents-to-software-factories]]*
*Last updated: 2026-11-26*
