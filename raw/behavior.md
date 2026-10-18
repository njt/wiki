---
url: https://swamp-club.com/behavior
date_fetched: 2026-10-18
---

# Behavior-Centric AI Design

The industry is undergoing a fundamental shift in how it builds AI systems, from early experimentation to at-scale production use. The current agent-centric approach says that we should provide the right context and tooling to a frontier model, and then allow it to decide the control flow of our applications. It has resulted in highly monolithic designs, where the resulting software is fragile, token hungry, hard to scale across the organization, and rarely delivers a positive ROI.

Instead, we should adopt a behavior-centric approach that uses frontier intelligence through a coding harness or AI agent to translate our intent into reusable behaviors that encapsulate their actions, own the data they produce, compose easily into workflows, and call out to frontier intelligence only when the job requires it. The results are agentic systems that are rapid to build, easy to reason about, deeply customizable, have clear security boundaries, and adapt to the social and technological challenges of scale.

## Agent-Centric AI Design

The agent-centric design philosophy has its roots at the very beginning of the modern AI era, with the release of ChatGPT in November of 2022. It made generative AI available as a general purpose chat application - by June of 2023, it was a public API with native, structured function calling. The core idea is simple - a user asks a question, and the LLM uses its reasoning to request more information from a tool, adds the answer to its context, and then decides if it has enough data to formulate the answer.

The LLM is the heart of the program, making decisions about control flow (what the program should do next). We have tried to scale this approach across many vectors: RAG for contextual search, skills to bundle context and scripts, MCP for making tools pluggable, MCP gateways for providing centralized security, model routers to make the choice of LLM pluggable, sub-agents to parallelize work, multiple agent memory schemes, harnesses to bundle it all together, etc. But regardless of what the industry adds, this core design philosophy remains the heart of it all.

A key consequence of this design are monolithic agents. In order to perform complex tasks, the right combination of context, tool calls, and model reasoning all must combine. If we continue to expand the capabilities the agent has within a single agent boundary, we end up with vertical scalability problems - we try to resolve them with things like tool search or code mode. If we break the agent up into smaller pieces, we end up with a distributed monolith - each piece tightly coupled to the others, but with a much more complex deployment and observability surface area.

No matter what trade-off you make, the LLM being responsible for control flow makes the resulting software fragile. The LLM decides what to do based on context - the user's input, system prompts, skills, tool call descriptions - and decides what to do probabilistically. For example, the above diagram describes a trip planning agent. The user’s expectation is that the itinerary always includes a hotel and a flight. The planning agent might decide not to call the hotel agent, causing the final output to no longer match the user’s expectations, or it might decide to omit the hotel based on budgetary constraints. The more an LLM is responsible for the control flow, the more the correctness of the resulting system relies on it making the right decision at every point - which cuts directly against the nature of the LLM.

To address this fragility, we make the system more robust by adding layers of rework and validation. For example, we might instruct the planning agent that it should always double check that the final itinerary has flights and a hotel, and if it is missing one, try the underlying hotel agent again. We might even decide to add yet another agent, whose only job is to validate the results of the planning agent, and run the entire loop again if it encounters an error.

This is the source of the dramatic cost of running complex agentic systems at
scale. Each new agent has to spend tokens on gathering its own context,
reasoning about its next steps, appending the responses to its own context, and
reasoning about its response. Then *those* agents require even more agents to
validate that the system as a whole didn’t subtly alter the user expectation. As
an engineer, we are stuck making a tradeoff between systems that are cost
efficient, performant, or resilient to transient failures we pass on to the end
user.

A unique property of these fragile systems is that they can be acceptable to the person who designed them. When they ask their agent to plan a trip to Japan, they know how the system was designed, and why it sometimes decides not to give them a hotel. They can easily overlook it - they know what they wanted, they know why it failed, and they can just try again until they get it right. It’s a papercut - a small, annoying moment in their day.

When we scale these systems up in an organization by adding more users with different expectations, or have them take on more and more responsibilities, these properties become unacceptable. On a social level, imagine this was a booking agent for travel within the enterprise - suddenly the end users would be far less forgiving of an incomplete itinerary (especially if they trusted the system a little too much and wound up on a business trip without a hotel)! On a technical level, the solutions to the fragility require an ever widening deployment, observability, and security footprint - making them harder and harder to maintain operationally.

This agent-centric design approach is the reason why many agentic AI systems fail to show a positive ROI: Gartner predicts that over 40% of agentic AI projects will be cancelled by 2027, and McKinsey reports that only 37% of the enterprises using AI can show any financial impact. Frontier intelligence is capable of solving a huge number of problems across our businesses, and it does - at the level of individual productivity, for example, in software development velocity. But as we scale those approaches up through the agent-centric lens, we commit ourselves to a death spiral: to make the systems resilient enough to be acceptable at the level of quality users demand, the costs balloon out of control, destroying any economic incentive we had to adopt them.

- of agentic AI projects will be cancelled by 2027
- 40%+
- of all enterprises using AI can show any financial impact
- 37%

## Behavior-Centric AI Design

In behavior-centric agent AI design, we focus on building a corpus of reusable, composable behaviors. Each behavior has typed, validated inputs and outputs; produces structured data that conforms to versioned schemas and is automatically indexed for search; and has its own isolated access to credentials. The platform enforces these properties at runtime rather than by convention or code review. The result is a building block that composes reliably because its contract is guaranteed, and can be maintained by frontier intelligence.

The above diagram shows the same trip planning agent as before, but built in a behavior-centric way. We first decompose our intent (“plan trips”) into a series of behaviors with methods - the first one calling Anthropic’s Haiku 5.5 to extract the destination city, budget, and dates from the user's initial prompt. This behavior writes its findings as structured data to the data layer, where it is automatically indexed for search. Hotel and flight behaviors then search within cities for given dates, also storing their results as structured data. The itinerary behavior chooses the correct flights and hotels from a list of options and generates the final itinerary, storing its progress and the final results as structured data.

These behaviors then get composed together into a “plan-trip” workflow, which calls the correct methods in order, populating the arguments for each method from the results stored as structured data. In this example, only the first step calls a frontier model; in more complex workflows, multiple behaviors might each call an LLM for the specific subtask that requires intelligence. Unlike agent-centric design, each LLM call is scoped to a single, well-defined job and never drives the control flow of the broader system.

To turn this behavior into an AI agent, we connect it to an LLM with a tool call as before. The difference is that now we’ve reduced the surface area to a single call (via a tool, MCP, or as a CLI command) - passing the user’s prompt into the plan-trip workflow. The workflow calls out to the correct frontier model, searches in parallel, picks the right choices algorithmically, and generates a standardized itinerary. If this was a more complex agent that needed to apply reasoning to decide which pathways to take to solve a problem, that reasoning would be encapsulated in a behavior whose inputs are the branches, and the outputs are the choices.

The fragility issues with agent-centric design are gone. Each method on a behavior can be invoked directly, with its arguments typed and validated at runtime. They produce structured data that must conform to known, versioned schemas. No tokens are spent at all on control flow - only on using a cheap frontier model to transform a free-form prompt into structured data (or perhaps return a hint for the agent to ask the user to provide any missing information). There are no validation steps needed after the first step in the flow because everything else is regular old deterministic code.

The scaling issues are eliminated. Each user who requests an itinerary will receive one, delivered in the same shape every time. Adding more functionality to the system has no impact on the deployment, observability, or security of the system - it’s just one more behavior that can be invoked from anywhere that needs it, and each workflow, method invocation, data access, or credential used can be tracked automatically with existing standards like OpenTelemetry.

Token consumption is dramatically reduced. For our simple trip planner, an
implementation using the behavior-centric design used 14x fewer tokens and costs
6x less per plan1. The savings are easy to attribute - the only call to
frontier intelligence in the behavior-centric design is extracting the initial
constraints for the trip, versus passing the results of each step back into the
LLM in order to orchestrate the next action. There is no longer an ROI death
spiral as scale is encountered - what worked for one user works for the whole
enterprise, at a fixed and understood cost.

- fewer tokens with the behavior-centric design
- 14x
- cheaper per plan
- 6x

The behavior-centric design also brings a unique benefit. Every behavior you create adds to a shared corpus of behaviors which can themselves be extended and reused in unique ways. When your DevOps team uses this pattern to manage infrastructure, the application development team can use those same behaviors to automate their application deployment process, and the SRE team can use them to help troubleshoot errors. By focusing on building repeatable, composable behaviors, you unlock the real value of enterprise-wide AI adoption.

## Swamp is the proof

Swamp is a platform for building AI systems using the behavior-centric design pattern. It plugs into any coding harness and provides the skills and CLI tooling required for it to use frontier intelligence to translate your intent into a behavior-centric AI system. As an example, here was the prompt for the trip planner:

*Build a workflow that plans trips using swamp. It should take a request like
“plan a trip to japan”, extract the parameters of the trip (like when I leave,
what city I’m going to), or fail with the missing parameters, then search hotels
and flights, and pick the best option based on budget and comfort.*

The harness then translated the prompt into the design above, built the search model for flights from scratch, reused an existing behavior for hotel search, and finalized the itinerary. It created all the behaviors, modeled the data, wrote the workflow, and tracked all the results. Instead of using frontier intelligence at runtime to decide what to do, we use it at build time to encode our intent into behaviors we can rely on.

Thousands of software developers have built production AI systems with Swamp, solving an incredible array of different problems. We’ve seen everything from software factories, infrastructure automation, and managing home labs to automating science lab experiments, controlling robot arms, sales lead scoring, and marketing management.

The pattern delivers measurable results in production. Teams running code review workflows through Swamp have seen an 8x reduction in token costs and a 3x improvement in execution speed when compared to the agent-centric approach. Production SaaS services have been built and launched in days, with their entire team empowered to evolve them. Marketing teams use it for self-service campaign targeting, turning campaigns that would have taken days to build into minutes. Our own team’s software factory enables individual developers to ship hundreds of times a week, resolving reported bugs within an average of four hours.

Behavior-Centric AI Design works. It solves all of the problems that agent-centric design causes, creates a shared corpus of capabilities that compounds over time, and ultimately enables you to deliver on AI projects that can’t justify their ROI any other way.

## Footnotes

- 
Agent-Centric solution (https://github.com/adamhjk/langchain-trip-planner) built using Anthropic Haiku 5.5 with LangChain and LangGraph, unoptimized, with Codex using GPT-6.1-Sol medium. It consumed 24,611 input tokens, and produced 944 output tokens. Behavior-Centric solution (https://github.com/adamhjk/swamp-trip-planner) was built using Anthropic Haiku 5.5 with Swamp, unoptimized, with Opus 5.5 medium. It consumed 1,183 input tokens and produced 680 output tokens. ↩
