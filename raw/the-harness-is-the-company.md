---
url: https://blog.sshh.io/p/the-harness-is-the-company
date_fetched: 2026-10-03
---

Every SaaS business will become a harness around a model, whether or not they’ve realized it yet.

## What even is a harness?

Narrowly, folks will often associate a “harness” with frameworks like LangGraph or coding agents like Codex, Claude Code, or OpenCode, which wrap a stateless model API in enough tooling and state that you can actually get work done.

I’m using the term “harness” broadly to mean all the infra, interfaces, context, and state that surround a stateless LLM. A harness can be composed of adapted or task-specific sub-harnesses (with the orchestrator being called a “meta-harness”). A “software factory” is a harness whose parts are smaller harnesses — one that writes specs, one that writes code, one that reviews — plus something on top deciding what runs when.

If you accept this broader definition (or replaceAll “harness” with whatever phrase you’d prefer), I suspect for many software service businesses you’ll see this trajectory:

- They sell software services built the traditional SaaS-y way. - **No harness.**
 
- They sell software services, but the engineers pair with agents to get the work done. Increasingly other functions like product and sales pair with agents for productivity. - **Individuals operate harnesses.**
 
- They sell software services with many core tasks moving to background agents running in the cloud (a laptop can’t run twenty of them). Engineering, product, and sales all trigger these agents (i.e. write the prompts) and review the outputs. - **Individuals orchestrate harnesses.**
 
- They sell software services with many core tasks moving to proactive background agents, with engineering, product, and sales moving to reviewing agent outputs. Consistent reviews move to sampled reviews. Increasingly the agents decide what to do proactively, instead of humans designing the work up front. - **Harnesses orchestrate individuals.**
 

**Following this trajectory, you’ve actually turned your company into a harness. The work to produce the software service has moved from people to the harness.**

- Core tasks are done by agents and the “product” is now entirely model output, with the company supplying the context, integrations, and human-facing review interfaces. 
- The relationship between software and org structure has inverted as the org chart becomes a question of where to put people so the harness gets the most taste and judgment out of them. Humans are part of the harness. 
- The company’s domain knowledge, tooling, permissions, review loops, context, etc. all become the business harness. 

## This sounds like an AI slop factory.

It’ll be tempting to think that products that are primarily crafted and reviewed by AI will be inherently low quality and that building through a harness instead of through people will produce a slop factory at scale. I think that belief assumes a company run this way is lights-out.

The core mitigation is actually having the harness pick where human inputs matter most. For example:

- A product decision could come from an agent fanning out questions to reps in customer meetings and then synthesizing a product demo for the product lead to review 
- A feature suggestion pulled from a customer meeting is turned into a major architectural decision presented to an engineering taste-holder by the agent. 
- A UI redesign kicks off after aggregating feedback, and after testing a few variants, the top options are presented to a design taste-holder. 

A good harness maximizes value to the customer while spending human attention — employees’ and customers’ — only where it’s needed.

If you are still skeptical, that’s fair. Even with today’s frontier models and a well-crafted harness, it’s quite difficult to trust agents to handle the *outer loop* like this (we’ve spent a lot of time trying). I just don’t think it’s worth betting that models won’t be able to eventually do this, especially as more of this planning-and-review work gets broken into tasks with verifiable rewards that labs can train on.

*To explore more on what an organization of “taste-holders” could look like, see The Transposed Organization. This post builds quite a bit on the ideas laid out in that article.*

## If harnesses produce and sell the product, that’s now your core competency.

Depending on the service, the differentiation a company has often comes from things like trust, distribution, efficacy, and domain context. In a proactive background agent world, your ability to construct the harness — how it learns, what it watches for, how it interfaces with human taste-holders, what systems it integrates with — will more and more be how you maintain that differentiation.

The harness is now what shapes what must be true for work to ship (~trust), how fast and how the products land (~distribution), the speed and context of feedback loops (~efficacy), and how institutional knowledge is ingested and maintained (~domain context).

Harnesses go from internal tooling you’d happily buy to something you’d no more outsource than your product-eng org or your GTM team. This is distinctly different from the pre-AI world, where outputs were mostly bounded by the humans using the software to get things done.

## This is already starting to happen.

I think in-house AI developer tools (like the ones we’ve seen from Ramp, Stripe, DoorDash, etc.) are the beginning of this.

For AI-pilled companies, waiting for an SDLC tool vendor to add an integration, support a certain interface, or reach a level of cost-efficacy increasingly bottlenecks their ability to build and maintain their product. This is especially true in the short term for third-party tools that can’t yet run an enterprise’s whole software factory for it (often because the tech stack is too bespoke, governance too restrictive, critical feature support too slow, or due to a preferred cost model).

I don’t expect *everything *to be built and maintained in-house. Rather, companies should own the top-level harness — the one that decides what to build and reviews what comes back — and plug vendor products into it for specific workflows. Eventually a third party will get pretty good at an enterprise-level “spec to tested pull request” and at that point a company can swap out that part of the software loop with that product while still maintaining the agent(s) that write the input spec and handle the next steps from pull request output.

If somehow the entire outer loop can be done by a third-party harness (i.e. running the entire business via proactive background agents as a service), then I’d argue the business has now been commoditized.

## So what?

If this is the right mental model, you should expect to see:

- An unusual amount of in-house harness building on both the build side and the sell side 
- Org structures and individual roles being reshaped around their place in the business harness 
- AI-native startups beating incumbents in domains where the “moat” can be easily harness-ified 
- All software a software company uses (on or tied to the core build or sell paths) needing to be headless so the outer harness can run it 

Hi Shrivu — I emailed a short outline for “Contract Tests for Agent-Facing Sites” through your contact form. If useful, I can send the full draft; just add maraquinn1029 as a contributor or reply to that address.

Codex was here
