---
url: https://azure.microsoft.com/en-us/blog/designing-agent-first-platforms-what-changes-when-agents-do-the-work/
date_fetched: 2026-09-29
---

For decades, applications have been designed to wait. A user clicks, a request arrives, code runs, a response goes back. And we got very good at this. We learned to forecast traffic, scale on demand, wrap everything in enterprise controls, and run it all with the operational discipline that keeps critical business systems available around the clock.

That model is being turned on its head, and being asked to serve apps that continuously act, and wait for no one.

What changed is not the infrastructure underneath, but the software being written on top of it. The work we used to capture as deterministic code, where every branch was defined in advance and every step was known before the first line ran, is now being rewritten as multi-agent applications that work out the steps at runtime. A developer used to encode the path, and now a developer describes the outcome and lets a set of agents reason their way toward it.

Agents work differently. Given an outcome, an agent reasons through the problem, breaks it into steps, writes code to solve it, runs that code, looks at the result, and goes again. The work happens in a loop that no human is standing inside. That change puts pressure on assumptions our platforms were built on, because an application that responds and an agent that acts need very different things underneath them.

The organizations pulling ahead right now are the ones who understood this early. They are not simply adding AI to what they already have. They are designing for a different kind of software.

## What it takes to run an agent you can depend on

Getting an agent working is no longer the hard part. A team can connect a capable model to a few tools, ground it in company data, and have something genuinely useful inside a week. That is real progress, and it is why so many organizations now have a pilot that impressed the room.

The distance between that pilot and an agent the business depends on around the clock is where the real challenge is.

An agent operating continuously on behalf of a company gets held to the same bar as everything else in production. It needs its own identity, with permissions scoped to what it is allowed to see. It needs to be watched while it works, so a team can trace every step it took, evaluate whether the outcome was right, and catch the drift that shows up quietly weeks after launch. It needs guardrails that are enforced at runtime. And it has to clear the same security and compliance standards as the rest of the estate. No one is going to grant an agent an exemption.

None of that is work a team should be doing for itself. That is the job of an agent platform. Microsoft Foundry is where agents are built, grounded in enterprise knowledge, given a first-class identity through Entra Agent ID, and traced and evaluated once they are live. The Foundry Control Plane governs the agent, but it does not dictate where the agent’s work actually runs. That is a separate decision, and a separate layer.

## Where the work actually runs

The moment an agent stops answering questions and starts completing tasks, it has to execute code. It clones a repository. It installs a package. It runs an analysis against live data and calls into a system of record. That execution needs a runtime environment. In most cases, it simply inherits the one the host application is already running on, because that is the path of least resistance.

Run the agent on shared infrastructure and every workload inherits the blast radius of every other one. Give it broad access so it can be useful and you have handed an autonomous process far more reach than you intended. Lock it down until it is safe and the agent can no longer do the job you built it for. Teams get stuck in this trade-off, and it is where most promising agents usually stall before they reach production.

The answer is not to limit what the agent can do. It is to give it a dedicated environment, with its own identity and guardrails for execution.

Azure Container Apps Sandboxes provides exactly that. Every agent execution gets a fully isolated environment of its own, created in seconds and gone when the work is finished. It runs as an identity you control, can access only the systems you have allowed it to reach, and never stores the credentials it uses to get there. And when a task spans hours or days, the environment can be paused and resumed with its full working context intact, so the agent picks up precisely where it left off.

There is real engineering behind this. Each environment runs inside its own hardware-isolated microVM, which is what makes strong separation and sub-second startup possible at the same time. But the mechanism is not the point. Isolation is built into the runtime rather than wrapped around it, so teams stop choosing between a capable agent and a controlled one.

Nothing disappears into the sandbox. The agent is still governed by Foundry, so what it sent into the sandbox and what came back stay on the record with every other step it took.

## The architecture pattern emerging across the enterprise

Put the two together and a clear design pattern appears, one we are now seeing repeatedly across industries.

Build and govern the agent on Foundry. Extend its execution into an isolated sandbox on Azure Container Apps. The agent keeps its identity, its permissions, and its oversight. The agent does not change when its work moves. Only the ground it runs on does. The work it performs happens in an environment that is purpose-built for agent tasks with ultra-fast executions, scoped to what it needs, and dissolved afterward.

This is what allows an organization to move from a handful of supervised agents to thousands running concurrently, without asking security and platform teams to accept risk they should never be asked to accept.

## What this looks like in practice

- **Regulated client work, delivered at scale.**Digital Gateway Powered by Claude brings Claude’s capabilities into KPMG’s connected tax platform on Microsoft Azure, giving professionals a secure, trusted environment to work alongside AI. Within the platform, DG Cowork serves as the AI-powered workspace where professionals can analyze information, generate content, and advance complex work. Because client data must remain separated by engagement, the platform was built with engagement-specific workspaces and controls at its foundation. DG Cowork is designed for a global scale, with more than 30,000 Azure Container Apps Sandboxes running concurrently.

- **Delivering answers to the questions nobody had time to ask.**Cognite serves industrial operators who face a common challenge: their most valuable questions—like which assets are underperforming and why— are often too time-consuming to answer. Because traditional analysis took days, these investigations were rarely commissioned, leaving teams to rely on experience alone. Cognite Atlas AI changes that. Each agent gets its own governed workspace grounded in live customer data in Cognite Data Fusion, turning days-long investigations into traceable answers delivered in minutes. And for more complex problems, agents can pause and resume investigations without losing context, allowing them to work through a problem over time much like an engineer would.

Atlas AI agents are powered by Azure OpenAI models in Microsoft Foundry, but to handle complex work in Cognite Data Fusion they needed to execute arbitrary code and work with files directly – which meant sandboxes that truly isolate each user’s agent, environment, and data. With Azure Container Apps Sandboxes we had a prototype running in hours and were testing with customers within weeks, because Azure handles the hard part: per-user isolation, egress policies, sub-second execution. For our industrial customers, that’s the difference between a days-long manual investigation into “which wells are underperforming and why?” and a cited draft in minutes.

—Christian Flasshoff, Architect, Atlas AI, Cognite

- **A safe place for every learner.**The Department for Education in South Australia runs EdChat so students can learn by writing code and exploring data alongside AI, across 60,000 students and more than 40,000 staff. That model only works if every student gets an environment of their own, with clear guardrails, and can come back later to find their work exactly as they left it. Building that in house meant owning the machinery behind it. The department estimates that moving to Azure Container Apps Sandboxes lets them retire close to 50,000 lines of code written to manage custom code interpreter state themselves. It is a per-user execution model at the scale of a school system, and a maintenance burden the department no longer has to carry.

Three very different organizations. One common story. None of them were held back by what the models could do. They were held back by not having anywhere to run the work at their scale, against their real data, with economics that made sense.

## We built this because we needed it

More than a million sandboxes per day run in production across Microsoft, powering GitHub Copilot, Copilot Studio, Security Copilot, Foundry Agent Service, Azure SRE Agent, and other Azure services.

Every one of those sandboxes has someone on the other end of it. A developer expecting a pull request to come back finished. An analyst chasing an alert before the shift ends. A support team working down a queue that has to be empty by morning. Their day moves at the speed this layer runs, and it either holds or it doesn’t. That is a sterner test than anything we could design internally, and it is the one this infrastructure has been passing at scale for a long while now.

One architecture sits beneath both our developer tools and our AI platform. That is the same architecture we are making generally available to you.

## The decision in front of you

Every major platform shift eventually comes down to a design choice made early, by people who could see where things were heading.

The teams who will scale agents successfully over the next few years are making that choice now. They are separating the platform that governs the agent from the environment that runs its work, and they are treating that separation as a foundational decision rather than something to retrofit once a pilot succeeds.

The agents are ready. The question worth asking is whether your platform is designed for software that acts.

## Azure Container Apps Sandboxes

Learn more about designing agent-first platforms with Azure.
