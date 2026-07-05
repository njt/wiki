---
url: https://kurtiskemple.com/blog/agentic-experience-design/
date_fetched: 2026-07-05
backfilled: true
---

# Agentic Experience Design

**The single biggest factor in whether an agent gets adopted or abandoned is experience design.**

In my role as head of developer strategy and platform experience for the Slack Platform at Salesforce, I have a unique vantage point—I’ve seen firsthand what admins will actually install in their organizations and which agent experiences actually lead to adoption rather than abandonment. I’ve distilled these adoption patterns into the following framework.

Agents introduce a genuinely new paradigm in human-computer interaction (HCI)—partial autonomy. Partial autonomy is defined as a system that can plan, take action within bounds, and is responsible for the outcomes of that plan. That’s what makes agents a new interaction layer—they introduce a fundamentally new relationship between human and system, one that restructures the entire interaction hierarchy.

This is part of a well-established pattern. Every time a new interface introduces a new paradigm—CLI, GUI, web, mobile—it creates a new experience design space with its own constraints, patterns, and principles. Each shift followed the same structure: the new layer emerged, old design principles didn’t fully transfer, and a new discipline had to be developed to account for the layer’s unique properties.

We’re in the middle of the partial autonomy paradigm shift now. The common agent failure modes we see today are a manifestation of old principles not transferring—agents don’t have their own place in the hierarchy because the hierarchy was never restructured to accommodate them. This is what makes agentic experience design a necessary and distinct discipline.

The agentic interaction layer requires a fundamental reevaluation of where agents fit in the HCI model. This new model must account for the changes in ownership of responsibility, planning, and authority that partial autonomy introduces.

## The Responsibility Model

Partial autonomy requires clear boundaries of responsibility at each layer of the interaction hierarchy. Previously, humans were responsible for everything above automation and execution—intent, judgement, planning, and outcomes. Workflows handled automation. Tools handled execution.

Partial autonomy changes that. With agents taking on planning and outcomes—responsibilities that previously belonged to the human—a new layer is created in the interaction hierarchy. The injection of this new layer between humans and workflows forces a redefining of the responsibility boundaries across the entire stack:

- **Human (Intent and Judgment):**The human provides the intent that drives the entire system and the judgment that governs it. Intent is what needs to happen. Judgement is the ongoing evaluation of whether what’s happening is right—whether the direction is correct, whether the tradeoffs are acceptable, whether the outcome meets the actual need. Judgment doesn’t get delegated. It’s the reason the human sits at the top of the stack.
- **Agent (Planning and Outcomes):**The agent translates human intent into a plan and is accountable for the plan’s outcomes. This is what distinguishes an agent from a workflow or a tool. A workflow executes a predetermined sequence. A tool performs a discrete function. An agent figures out what to do given a goal and is responsible for what happens as a result. Planning requires understanding the goal, the constraints, and the available resources. Accountability for outcomes means the agent’s performance is measured by whether the results serve the human’s intent, not by whether it followed the steps correctly.
- **Workflow (Automation):**Workflows execute structured, repeatable sequences. They don’t decide what to do—they carry out a defined process. Their value is reliability and consistency. A workflow that sends a weekly report, processes an intake form, or deploys a build artifact does the same thing every time, and that predictability is the point.
- **Tool (Execution):**Tools perform discrete functions. They take an input, produce an output, and have no memory or judgment about why they were invoked. A function that resizes an image, an API that sends an email, a database query—these are tools. They do one thing, and they do it on demand.

The responsibility model redefines the boundaries of responsibility across the stack to accommodate the agent layer. Because autonomy must align with responsibility, the distribution of autonomy across the system must be redefined as well.

## The Autonomy Model

Each layer in the responsibility model requires a specific degree of autonomy to fulfill its role. Before agents, the hierarchy was straightforward—humans had full autonomy over intent, judgment, planning, and outcomes. Everything below them was deterministic or invoked on demand. Workflows ran their sequences. Tools executed their functions.

With agents now responsible for planning and outcomes, they require a minimum amount of autonomy to carry out those responsibilities. Because the scope of planning and outcomes an agent is responsible for can expand over time, its autonomy must be bounded and not rigid. The autonomy model defines the degree of autonomy each layer requires to fulfill its role in the responsibility model:

- **Human (Full Autonomy):**They set direction for the entire system, override agent decisions when their judgment says the outcomes aren’t serving their intent, define the boundaries of their autonomy, and evaluate whether the outcomes delivered actually meet the need. Full autonomy means the human retains the ability to intervene at every level of the hierarchy at any point.
- **Agent (Bounded Autonomy):**They can plan, adapt, and make decisions within the scope they’ve been given. But the boundaries are real. An agent can choose how to approach a research task, but can’t decide to publish the results without approval. The bounds exist because the agent is accountable for outcomes, but doesn’t own the judgment about whether those outcomes are acceptable—the human does.
- **Workflow (Deterministic Autonomy):**They execute their defined sequence without deviation. They don’t adapt, they don’t decide, they don’t choose between approaches. If a workflow encounters something outside its sequence, it stops or escalates. This is a feature, not a limitation—deterministic behavior is what makes workflows trustworthy for repetitive tasks.
- **Tool (Zero Autonomy):**They respond to invocations. They don’t initiate action, maintain state, or make decisions. They execute a function and return a result.

Responsibility and autonomy define the environment within which agents plan and deliver outcomes, but defining that environment is not enough on its own. For agents to actually carry out that role, they need context to plan and authority to take action.

## Planning Requires Context

Context is the informational environment that makes coherent planning possible over time. As the scope of outcomes an agent is responsible for expands, context gives the agent the continuity it needs to keep planning coherently across a larger body of work.

Context includes the goal, constraints, decisions already made, the current state of the work, the relevant history, and the open questions that still shape what should happen next. It is not just prior conversation or retrieved material. It is the working environment the agent plans within.

Without that environment, an agent can only react to the immediate turn in front of it. Local reactions can sometimes produce useful outputs, but they are not the same thing as coherent planning. Planning requires continuity: the ability to stay oriented as the work unfolds, dependencies accumulate, and the task’s state changes over time.

This is why drift is such a central failure mode in agentic systems. Without actively managing context over time, the agent’s understanding of the user’s intent can drift, and its view of the work’s state can become stale or incoherent. Those risks grow as interactions become longer, more complex, and more consequential.

Context management is the practice of maintaining the informational conditions for coherent action over time. The design problem is not just remembering more. It preserves relevance, structure, continuity, and history so the agent can keep planning toward the right goal, within the right constraints, based on the right understanding of the work’s current state and the user’s intent.

Because context shapes planning, it becomes part of the basis for justifying outcomes. Governing an agent, therefore, means governing not just what actions it can take, but what information, constraints, and prior decisions it is allowed to plan from. In that sense, context management is also an accountability surface.

Context provides the continuity that makes coherent planning possible over time. But coherent planning alone is not enough to produce outcomes; the agent also needs authority to act.

## Authority Enables Outcomes

Authority is what bridges the gap between having a plan and achieving outcomes. But because authority allows an agent to take autonomous action, it also introduces risk. The design problem is not simply whether the agent can act, but how that authority can be exercised without displacing human judgment.

For authority to remain aligned with human judgment, the agent’s actions have to stay transparent, steerable, and resilient under failure:

- **Transparency:**Users need to understand what the agent is doing, why it is doing it, and the consequences of its actions. That includes being able to distinguish agent actions from human actions, trace outcomes back to the inputs and constraints that shaped them, and remain oriented during long-running work. Authority that cannot be clearly seen cannot be trusted to take meaningful action.
- **Steerability:**Users need to be able to intervene in the agent’s work while it is still underway. Meaningful steering is not limited to approval or denial at the edges. It includes the ability to pause, redirect, constrain, or stop the work when human judgment needs to reassert itself. Delegated action remains aligned only when the human can correct the trajectory before the outcome is final.
- **Resilience under failure:**The agent’s authority has to remain usable even when the task cannot be completed cleanly. When an agent gets blocked, partial progress should remain legible and useful rather than collapsing into a dead end. Safe authority depends on the quality of the handoff as much as on the quality of successful execution.

Authority also requires mechanisms for error correction and explicit boundaries on scope:

- **Reversibility:**Important actions should be undoable wherever possible. When users know they can recover from a mistake, the cost of delegation drops, allowing them to let the agent operate with greater confidence. Reversibility does not remove the need for caution, but it changes authority from a high-risk leap into a manageable extension of the user’s workflow.
- **Guardrails:**Authority needs explicit boundaries on which actions are available, which resources can be affected, and when expanded scope requires an explicit grant. For admins, guardrails are where organizational policy enters the system. They make authority bounded rather than ambient.

All of these conditions depend on the agent’s state being inspectable. Without a visible, queryable state of the work, transparency is reduced to narration, and steering becomes guesswork, while failure recovery, reversibility, and guardrails become much harder to trust in practice. Inspectable state is what turns those conditions from design intentions into working controls.

The agent cannot deliver outcomes without authority to act. But authority that cannot be seen, steered, or recovered from removes the human’s ability to exercise judgment and intervene. When that happens, trust breaks down and authority contracts—the agent is restricted to a narrow scope of outcomes. The goal is the opposite: authority that can expand as the agent earns trust, taking on larger and more consequential work over time.

## Progressive Authority Expands the Scope of Outcomes

Progressive authority is the mechanism by which the scope of outcomes an agent is accountable for expands over time. As the agent demonstrates competence with a given action, it is granted expanded autonomy—taking on larger, more consequential work.

An agent that has consistently delivered accurate research summaries within a defined scope has earned the trust required to take on broader research with less oversight. That trust is built action by action—the agent demonstrates it can handle a given action responsibly, and expanded autonomy is granted for that action as a result. The same principle governs how trust works between people: it accumulates through demonstrated competence and is extended through deliberate decisions, not assumed.

This operates at two levels simultaneously:

- For users, the agent earns trust through consistent, reliable performance with each action within defined bounds. As that trust builds, the user can choose to grant expanded autonomy—taking on higher-stakes actions, broader scope, less oversight. That grant is always an active human decision. The agent never assumes progressive authority.
- For admins, progressive authority means starting with a narrow set of agent capabilities and expanding access as trust in the system builds. A phased deployment—read-only access first, then write access for low-risk actions, then broader operations—maps onto how organizations actually adopt new tools and gives them a clear on-ramp to full deployment.

Progressive authority completes the chain—responsibility defines what the agent owns, autonomy defines the boundaries within which it operates, context enables coherent planning, authority enables outcomes, and progressive authority allows the scope of those outcomes to grow as trust is earned and autonomy is granted.

## Conclusion

Agentic experience design is a new discipline because agents represent a new interaction layer—and new interaction layers have always demanded new ways of thinking about design.

The discipline starts with a responsibility model. Humans own intent and judgement. Agents own planning and outcomes. Workflows own automation. Tools own execution. Responsibility sets autonomy: the freedom each layer gets is determined by what it’s responsible for.

From there, a causal chain connects the remaining principles. Context enables planning—without it, the agent can’t fulfill its core responsibility. Authority enables outcomes—without it, the agent has a plan it can’t execute. Progressive authority expands the scope of outcomes over time, as the agent demonstrates competence and earns trust from users and admins alike.

Throughout, the discipline must account for every persona the layer touches—users who interact directly with agents, developers who build agentic experiences on platforms, and admins who govern them across organizations. A system that works for one persona but fails the others won’t survive contact with production.

The design posture is consistent: ship with narrow, inspectable, reversible defaults. Make the authority structure visible. Invest in context management so the agent stays coherent over multi-step, long-running work. And let authority expand progressively as trust is earned. Constraints aren’t limitations—they’re what make the product usable at an organizational scale.
