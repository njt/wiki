---
url: https://www.oreilly.com/radar/how-to-build-reliable-ai-agent-systems-for-production/
date_fetched: 2026-10-11
---

The following article was originally published on theAgentic AI Foundation blog siteand is being republished here with the author’s permission.

A customer asks your company’s AI agent to update the shipping address on account 123. The agent relies on a support ticket with a typo and updates account 132 instead. The customer relationship management system reports that the update “succeeded.”

The API successfully completed the authorized request, but because it had no way to know which account the customer approved versus what was intended, it saw no error.

Teams often try to improve reliability by tuning the prompt or choosing a stronger model. But the model did exactly what the surrounding system allowed because the system treated the agent’s decision as the final authority. Prompt tuning and stronger models can improve the agent’s responses, but can’t provide guarantees for actions.

## Put each decision in a layer that can enforce it

The model should propose an action, while a policy service decides whether the action is allowed. The execution layer can then perform the action within those limits.

Now consider that same agent, but with a policy service in place. The agent would propose updating an account after reading a support ticket. Before the update runs, a policy service can compare the request with the change the user approved. If the request falls outside those limits, the service can reject it even when the agent sounds confident.

After the policy check, the system can save an audit record that connects the request to the result and identifies the policy version used for the decision.

This separation gives each part of the system a job it can handle. The model interprets the request, while deterministic software enforces the conditions that must always hold.

However, policy enforcement depends on the information used to request an action. A policy check can’t correct a decision that was built on context the system should never have trusted.

## Treat agent context as untrusted input

Agents receive instructions from more places than the user’s current prompt. A coding agent may read a `SKILL.md` file or persisted memory from a previous run. Each source can influence what the agent does next.

Stored instructions are useful, but their source may be stale or malicious. Another agent may have written the memory. A downloaded skill may contain instructions that expose credentials.

Therefore, stored context needs provenance. The system should record who wrote the context and when, then verify that linked files still match the versions the writer used. A risk label can also tell the runtime whether a piece of context may guide an answer or authorize a write.

Warnings help, but they don’t remove the risk. In controlled testing described in “Trustworthy Context Is Untrusted By Default,” Shub Argha reports that a trust preamble reduced one class of context contamination from 88.8 percent to 33.3 percent. The remaining failures show why a warning should sit alongside technical checks.

Once a team can tell where context came from, the next decision is what an agent may do with it.

## Give the agent only the authority required for the task

OAuth scopes provide a useful boundary, but a broad `write` scope still leaves a large decision to the agent. The token may allow the agent to update any record even though the user approved one field on one account, as we saw in the opening example.

A narrower capability can represent the exact action the user approved. For example, the runtime could issue a capability that permits one update to the shipping address on account 123. The capability expires after the update, so the agent cannot reuse it for another customer.

Narrow capabilities also apply to local execution. An agent that needs to format one file doesn’t need unrestricted shell access. The runtime can provide a tool with the necessary input and keep other commands unavailable.

As a result, a prompt injection has less authority to work with. The agent may still request the wrong action, but the runtime can reject anything outside the capability it received.

Narrow permissions also make audit records easier to understand because the record contains the authority granted for that specific action. When authority lives in a broad token or a long prompt, an operator has to reconstruct what the agent was supposed to do after the failure.

Even narrow authority doesn’t require an agent to act. A reliable system also needs clear conditions for when the agent should stand down.

## Make stopping part of normal operation

An agent can cause damage even without calling a sensitive tool. It can post a wrong answer to a customer or keep replying after a human has taken over.

The runtime should make restraint part of the workflow. If the agent’s confidence falls below a set threshold, the runtime can route the support ticket to a human representative. A reply from the human can then cancel any pending response from the agent.

Confidence checks and rules that stop the agent when a human takes over belong in the routing and execution layers. The model shouldn’t make either decision on its own. Similarly, a kill switch must stop an agent even when the model is in the middle of a plan.

Stopping safely solves one part of reliability, but long-running agents also need a plan for failures that occur after valid work begins.

## Preserve state so the system can recover

Consider a browser agent that has filled out most of a form when the page changes. A retry that starts from the beginning could submit an earlier step twice, while a retry that guesses where to continue may skip a required field.

The execution system should record each confirmed step and attach an idempotency key to any action that must happen once. The key is a unique identifier that tells the server a retry belongs to the same action. Then, when the workflow resumes, the system can continue from the last confirmed state without repeating a completed action.

Agent sandboxes create a related problem. Keeping every sandbox running during long idle periods wastes resources and keeps execution environments available longer than needed. Hibernation can reduce both concerns, but only when the wake-up process restores the required state and handles a failed resume.

Recovery becomes much harder when the system can’t explain what happened before the interruption.

## Keep evidence that people and agents can inspect

An execution log should connect the context an agent received to the action it requested. It should also show which policy allowed the action and what result came back.

Teams can use that record during an incident, but the agent can also use it during later work. For example, an agent that can read the reason behind a previous code change doesn’t need to infer intent from the final diff alone.

Context graphs offer one way to preserve those relationships across time. A graph can connect a tool call to the policy that governed it. The same record can link the source context to the outcome. Later, a person or agent can query the relationships to understand why the system made a decision.

The record still needs limits. Secrets shouldn’t be copied into an audit trail, for example, and retention rules should match the data involved.

A deliberate record is safer than relying on a conversation transcript and hoping it contains everything an operator will need.

A reliable agent system can explain why an action was allowed and resume safely when that action fails. Enforceable policies and recorded state provide those guarantees around the model.

## Keep the controls portable

Many organizations will use more than one model or agent runtime. When each agent carries its permissions inside a prompt, the rules can drift as teams add models and tools.

MCP gives clients and servers a shared way to describe and call tools. Teams can use that common tool surface to enforce authorization at the server or gateway, regardless of which model requested the action.

A shared context format can also preserve provenance when a workflow moves between agents. Open specifications provide consistent interfaces, while each deployment remains responsible for its policies and enforcement.
