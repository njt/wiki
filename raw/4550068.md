---
url: https://techcommunity.microsoft.com/blog/azure-ai-foundry-blog/stop-letting-agents-run-the-workflow/4550068
date_fetched: 2026-09-16
---

## A design pattern for deterministic multi-agent automation on Microsoft Foundry

A user submits a ticket: *"Please grant temporary admin access to the finance reporting database for month-end close."*

A five-agent system handles it beautifully. A triage agent classifies the request. A policy agent checks entitlements. An approval agent decides a manager needs to sign off. An execution agent calls the identity API. A communication agent updates the ticket. The demo gets applause.

Three weeks into production, the same system grants a contractor standing admin access to a production finance database for **30 days instead of 8 hours** because the classifier read "month-end close" as a recurring need, the approval agent inferred consent from an earlier message that said "manager aware," and the execution agent filled in a duration parameter the payload never specified.

Nothing hallucinated. Every agent did roughly what it was asked. The system still failed, because **no single component owned the process**.

This is the most common way enterprise multi-agent systems fail, and it is not a model problem. It is an architecture problem. This article describes the pattern I use to prevent it:


Deterministic Spine, Agentic Leaves - Use deterministic workflow control for the business process. Use agents only as bounded workers inside controlled steps.

This is not an observability pattern that helps you find the failure afterwards. It is a design-time pattern that makes the failure unrepresentable.

**What you'll get from this article:** a diagnostic for whether your system has this problem, the four-layer pattern, a reference architecture on Microsoft Foundry, the schema contracts that make it enforceable, and a six-step implementation checklist.

## 1. The failure mode: the agent quietly becomes the process owner

Dangerous multi-agent systems rarely fail loudly. They usually look like they are working.

A ticket gets created. A handoff happens. A subagent responds. A tool is called. The workflow advances. But **the path is not guaranteed.** The next step depends on what the model decided in that moment, based on prompt wording, context window contents, prior messages, tool descriptions, and sometimes stale state.

Run the same request twice and you can get two different process paths. In a brainstorming assistant, that is a feature. In an approval workflow, it is an unlogged control failure.

The specific failure modes are operational, not semantic:

| 
 | 
 | 
| Misclassification cascade | A wrong label at intake silently changes every downstream decision | 
| Inferred approval | The model treats conversational context as an approval record | 
| Parameter drift | The right API is called with a wrong or invented value | 
| Concurrent writes | Two agents update the same record, producing inconsistent audit history | 
| Retry amplification | A replayed action creates duplicate approvals, tasks, or grants | 
| Circular delegation | Agents hand off to each other without converging | 
| Silent subagent failure | A leaf fails, returns prose, and the parent proceeds anyway | 
| Bypassed parent | A subagent replies directly to the user, creating an unapproved side path | 

None of these are hallucinations. All of them are workflow-control problems. They happen because the system has no single deterministic owner of **state, transitions, permissions, and irreversible actions**.

#### A quick diagnostic

Ask these four questions about any agentic system you are about to ship:

- If I run the same request 100 times, is the **sequence of business states**identical every time?
- Can I name the **one component**that decides whether a transition is allowed?
- Is approval a **blocking gate in code**, or an instruction in a prompt?
- If a tool call is replayed, does the business system change **twice**?

If any answer makes you uncomfortable, the model is not your risk. Your control plane is.

## 2. Microsoft's guidance already draws this line

This is not a contrarian take. It is the documented design distinction in Microsoft Agent Framework.

The framework separates two things deliberately. **Agents** are for open-ended work: the task is conversational, you need autonomous tool use and planning, and a single model call with tools is sufficient. **Workflows** are for defined processes: the steps are well-defined, you need explicit control over execution order, and multiple agents or functions must coordinate. As the documentation puts it, Agent Framework workflows *"define explicit, inspectable execution paths for coordinating code, agents, state, events, and human input."*

Explicit and inspectable are the two words that matter for regulated enterprise processes.

The wider industry has converged on the same boundary, which is a useful signal that this is architectural rather than vendor-specific. Anthropic distinguishes **workflows**, *"where LLMs and tools are orchestrated through predefined code paths,"* from **agents**, *"where LLMs dynamically direct their own processes and tool usage."* LangGraph's documentation states that *"workflows have predetermined code paths and are designed to operate in a certain order"* while *"agents are dynamic and define their own processes and tool usage."* OpenAI's orchestration guidance separates **handoffs**, where control moves to a specialist, from **agents as tools**, where a manager retains ownership and calls specialists as bounded capabilities.

Four independent teams, one conclusion: **when the process must be trusted, the process must be predetermined.**

The design decision follows directly:


Use agents for intelligence. Use workflows for control. Use policy for authority. Use tools for execution.

## 3. The pattern

The core idea fits in one sentence:


The business process is a state machine. Agents are workers inside the states.

The **deterministic spine** owns workflow state, allowed transitions, approval boundaries, retry behaviour, idempotency, timeouts, tool execution, audit events, and final action authority.

The **agentic leaves** do bounded reasoning work: classify, extract, summarise, recommend, draft, compare, validate, explain, enrich.

The boundary is absolute:

- An agent can **recommend**the next step. It cannot**move**the workflow.
- An agent can **draft**an approval summary. It cannot**approve**.
- An agent can **prepare**an API payload. It cannot**execute**it.
- An agent can **identify**a missing control. It cannot**invent**one.

This is the difference between *AI automation* and *agentic business process engineering*.

## 4. Implementing it on Microsoft Foundry

Microsoft Foundry gives you the building blocks. The architecture decision is how you assemble them. Resist the instinct to build one "super agent" that owns everything, and implement four layers instead.

#### Layer 1: The workflow spine

This is the deterministic owner of the process. Implement it with **Microsoft Agent Framework workflows**, **Azure Logic Apps**, **Durable Functions**, or another deterministic orchestration runtime, depending on your engineering model.

One architectural principle matters more than the specific runtime: **the design centre should be a versioned workflow definition or a code-first orchestration layer, **something you can diff, review, test, and roll back. Visual authoring surfaces are excellent for prototyping, but authoring surfaces evolve and get deprecated. A workflow definition under source control survives that; a canvas-only design does not.

Define your states explicitly:


Submitted → Classified → PolicyChecked → ApprovalRequired → Approved → PayloadPrepared → Executed → Verified → Closed, plus Rejected and Exception.

Then give every transition a deterministic guard:

```
Submitted → Classified
  requires: classification_result.schema_valid == true
Classified → ApprovalRequired
  requires: risk == "high" OR action_type == "privileged_access"
ApprovalRequired → Approved
  requires: approval.status == "approved"
            AND approval.approver_id is valid
            AND approval.timestamp exists
Approved → Executed
  requires: idempotency_key exists
            AND execution_lock acquired
            AND payload.validated == true
```
The workflow owns these transitions. Not the agent.

#### Layer 2: Bounded agent leaves

Each leaf does exactly one job, with one narrow context and the minimum permissions to do it.

| 
 | 
 | 
 | 
| Intake classifier | Classify the ticket and extract fields | Approve or execute | 
| Policy explainer | Explain why a policy applies | Override the policy | 
| Payload drafter | Produce a structured JSON payload | Call the API directly | 
| Exception summarizer | Summarize a failure for human review | Retry execution independently | 
| Closure writer | Draft the final ticket update | Change workflow state | 

In Foundry these can be **prompt agents** when the task is configuration-driven, or **hosted code agents** when you need custom logic, state handling, or a specific execution model while still using Foundry models and tools.

**The contract is the control.** Any leaf whose output feeds the spine must return structured, schema-validated output, never free-form prose:

```
{
  "request_type": "privileged_access",
  "business_justification_present": true,
  "target_system": "finance_reporting_database",
  "requested_duration_hours": 8,
  "risk_level": "high",
  "requires_human_approval": true,
  "confidence": 0.91,
  "missing_fields": []
}
```
The workflow validates that schema. If validation fails, the workflow requests clarification or routes to exception handling. It does not let the agent improvise. The rule is simple: **if the spine cannot validate the output, that output cannot influence the next transition.**

#### Layer 3: Policy and guardrail gates

Business policy and AI safety are different controls, and conflating them is a recurring design error.

**Foundry guardrails and controls** address AI safety and security risks: unsafe content, prompt injection, personal information exposure with intervention points at user input, model output, and, for agents, tool call and tool response. 

But **do not encode enterprise approval logic in prompts or safety filters.** Business policy belongs in deterministic controls: Azure Policy, policy-as-code, a rules engine, service catalogue rules, entitlement policies, an approval matrix, ITSM workflow rules, or domain APIs.

The policy gate answers questions a model should never be trusted to answer alone:

- Can this request proceed without approval?
- Who is allowed to approve it?
- Is this action permitted for this user?
- Is the target resource in scope, and the duration within policy?
- Is this a break-glass scenario?

An agent can *explain* the policy. The policy system *decides*.

#### Layer 4: The controlled action plane

This is where real-world change happens, so this is where the architecture must be strictest. Granting access, closing incidents, updating records, triggering deployments, approving procurement, all of it goes through controlled tools, never through free-form agent autonomy.

Foundry Agent Service supports built-in tools, custom functions, and MCP servers, with authentication via key-based access, Microsoft Entra managed identity, OAuth On-Behalf-Of, or unauthenticated access where appropriate.

Wrap every action in a tool that enforces an explicit input schema, required fields, an idempotency key, caller identity, an approval reference, a correlation ID, a retry policy, a timeout, and deterministic success/failure responses.

The execution tool must reject this:

```
{
  "action": "grant access",
  "details": "finance database admin for a while"
}
```
And require this:

```
{
  "ticket_id": "INC1234567",
  "request_id": "REQ98765",
  "target_system_id": "finance-reporting-db-prod",
  "principal_id": "user-guid",
  "role_id": "temporary-admin",
  "duration_hours": 8,
  "approval_id": "APR456789",
  "idempotency_key": "REQ98765-finance-reporting-db-prod-temporary-admin",
  "requested_by": "user-guid",
  "approved_by": "manager-guid"
}
```
If the payload is incomplete, the tool rejects it. The agent does not get a second chance to guess.

Make tools boring and strict: validate input, reject ambiguity, return structured errors, enforce authorization, support idempotency, log correlation IDs, and **never infer missing business data**. Enterprise reliability is won here, in unglamorous code.

## 5. Five design mistakes to correct now

Most failed multi-agent prototypes are not under-instrumented. They are **under-designed**. The five most common corrections:

| 
 | 
 | 
| Start with agents and ask  | Start with the business process and ask  | 
| Add a supervisor agent and rely on instructions to enforce control | Add a deterministic workflow; let agents operate only inside explicit boundaries | 
| Give several agents overlapping tools and shared context | Give each agent a narrow job, minimum context, minimum permissions, one typed contract | 
| Rely on  | Make approval a transition guard that cannot be bypassed | 
| Treat tool calls as agent actions | Treat tool calls as controlled business operations with schemas, idempotency, and policy checks | 

Add a specialist agent only when it materially improves capability isolation, policy isolation, prompt clarity, or trace legibility: not simply because a task *can* be split.

## 6. A six-step implementation checklist

- **Map the process before you name a single agent.**For every step, classify it and assign an owner:- **Step type**- **Example**- **Owner**- Deterministic - Check the approval matrix - Workflow spine - Agentic - Classify the ticket - Agent leaf - Human - Approve privileged access - Approver - Action - Grant access - Controlled tool - Verification - Confirm access expires - Workflow spine - If a step must be reproducible, auditable, or policy-bound, it belongs in the spine. 
- 
**Define the state object explicitly.**Every workflow needs durable, inspectable state:`{ "workflow_id": "", "ticket_id": "", "current_state": "", "request_type": "", "risk_level": "", "required_approvals": [], "approval_status": "", "action_payload": {}, "execution_status": "", "verification_status": "", "exception_reason": "", "correlation_id": "" }`No agent should hold private state that determines the business outcome. 
- **Make every agent contract testable.**For each leaf, specify the input schema, output schema, allowed tools, forbidden actions, confidence handling, error behaviour, and escalation behaviour, then build an evaluation set per leaf, not per system.
- **P**- **ut approval before execution, not after explanation.**Human-in-the-loop is not a courteous notification after the fact; it is a blocking gate before sensitive execution. Agent Framework orchestrations support this through approval-required tools and request-info interactions. Use it for privileged access, production changes, financial approvals, customer-impacting actions, security exceptions, external communications, and any irreversible update.
- **Verify against the system of record.**A successful API response is not proof. Read the state back and confirm the change matches intent, including that time-bound access actually expires.
- **Choose the implementation path by risk, not by how "agentic" it feels.**- **Requirement**- **Recommended path**- Simple bounded agent on a managed runtime - Foundry prompt agent - Custom agent logic or framework code - Agent Framework with Foundry models and tools - Code-first production orchestration - Agent Framework workflow on managed compute - Visual business-process orchestration - Azure Logic Apps calling Foundry agents - Cross-agent interoperability - Agent-to-agent delegation where lightweight handoff is sufficient - Enterprise safety controls - Foundry guardrails plus deterministic policy gates 

## 7. The production boundary

The next generation of enterprise automation will not be won by the team with the most agents. It will be won by the team that knows **where not to use one**.

Multi-agent systems are genuinely powerful when they decompose expertise. They become dangerous when they replace workflow control. For tickets, approvals, onboarding, incident response, procurement, and finance operations, the boundary has to be explicit:

**The workflow owns the process.

The policy owns the permission.

The tool owns the action.

The agent owns only the reasoning task it was explicitly assigned.**


That is the production boundary. That is how multi-agent automation becomes safe enough for enterprise scale.

And that is the shift customers need now, not more autonomous agents, but **controlled autonomy**: deterministic where the business requires trust, agentic where intelligence creates value.

## Try it this week

Pick one workflow you have already automated with agents, and do three things:

- **Draw its state machine on one page.**If you cannot, the process has no owner.
- **Find every irreversible action**and confirm a deterministic guard sits in front of it — in code, not in a prompt.
- **Replay one tool call twice**in a test environment. If the business system changes twice, add an idempotency key today.

Then build it properly:

- **Microsoft Foundry**—- __learn.microsoft.com/azure/foundry__
- **Foundry Agent Service overview**—- __learn.microsoft.com/azure/foundry/agents/overview__
- **Microsoft Agent Framework**—- __learn.microsoft.com/agent-framework__·- __github.com/microsoft/agent-framework__
- **Orchestration patterns**—- __sequential, concurrent, handoff, group chat, magentic__
- **Human-in-the-loop workflows**—- __learn.microsoft.com/agent-framework/workflows/human-in-the-loop__
- **Guardrails and controls in Foundry**—- __learn.microsoft.com/azure/foundry/guardrails__
- **AI agent orchestration patterns (Azure Architecture Center)**—- __learn.microsoft.com/azure/architecture/ai-ml/guide/ai-agent-design-patterns__

**I'd like to hear from you.** If you have shipped a multi-agent workflow into production, tell me in the comments where you drew the line between the spine and the leaves, and where that line turned out to be in the wrong place.
