---
url: https://www.oreilly.com/radar/operating-mode-as-runtime-state-a-contract-for-enterprise/
date_fetched: 2026-09-13
---

During a service incident, a customer-remediation workflow is moved onto an emergency route because the situation is critical and the team needs a fast resolution. Approvals are shortened, a priority queue is opened, and an on-call agent is cleared to use an alternate procedure until the service recovers. The incident ends, but the route stays active for a small customer segment after everyone has moved on.

The emergency route itself was fine. Production systems need emergency routes, and a human approved this one. The trouble is that the route now runs without a live incident, an owner, or an expiry condition. A controlled exception has quietly settled into the platform’s standard runtime behavior.

This article is about that execution layer: the temporary operating state that lingers in runtime behavior. Temporary authority is easy to grant under pressure and hard to retire once the pressure lifts. An enterprise agent shouldn’t have to guess from prompts or conversation history whether it’s running under normal, incident, or recovery conditions. That state should be handed to it as authoritative runtime input, the way platforms already hand over identity, tenant, environment, and permissions.


An exception is safe while the incident that justified it is live. It becomes dangerous the moment the platform can no longer show the incident has ended.

**The problem is unmanaged exception state**

Exception drift is what happens when temporary exception behavior outlives its authorized scope, authority, or duration, and emergency accommodations settle into normal execution. The drift is usually quiet: a routing rule that stays reachable, an approval shortcut that survives closure, a tool permission that keeps shaping execution after the triggering condition has passed.

Enterprises already have the human machinery for this. Incident management defines abnormal conditions, change control governs deviations from standard practice, and postincident reviews confirm that temporary measures have closed.1,2 The gap is architectural. Most agent platforms still treat organizational operating state as something outside the runtime rather than an input to it.

Once an accommodation proves useful, it fades into an invisible operating state. The routing rule stays enabled, the shortened approval path stays reachable, and the temporary queue keeps taking work. No dramatic model failure is required, only a platform with no reliable way to close runtime state.

The practical question to ask is “Which operating mode is active for this workflow, this user, this segment, at this moment?” When the answer is left implicit, every agent, workflow, and tool gateway invents its own. Discussions of agent architecture tend to dwell on capabilities such as models, tools, and orchestration, yet production behavior depends just as much on runtime context: identity, tenant, environment, permissions, session state, policy, and tool access. One piece of that context is usually absent. The runtime knows who is acting and what they may do but not whether the organization is under normal conditions, incident response, recovery review, or a declared exception. Exception drift begins in that blind spot.

**The exception lifecycle**

Organizations move through a predictable sequence: normal operations, incident declaration, a temporary exception window, closure review, and return to normal. Each stage answers a question, from why the exception exists and who authorized it to what evidence shows it’s no longer shaping execution. Most enterprises handle the front of that sequence well; they’re practiced at declaring incidents and authorizing workarounds. The hard part is the retirement: proving that the exception behavior actually disappeared.

Declaring an exception is loud. Retiring one is quiet, especially when the workaround improved throughput or helped the team recover faster. That asymmetry is where drift lives, because an incident can be closed on paper while emergency routing, override policies, or alternate workflows keep influencing execution. From the platform’s point of view, the lifecycle should close only when it can show that every exception path has been retired or formally adopted through change governance.

**Why this matters more for AI agents**

Agents raise the stakes because they act. Rather than sitting in a config file, they select tools, trigger workflows, coordinate with other agents, and adapt their paths at runtime. An accommodation introduced during an incident can spread through routing, tool use, approval paths, and downstream agents. A traditional exception stays legible in a runbook or workflow definition; an agent can carry the same exception along many paths at once, which makes it harder to find and retire.

For engineers, the missing layer is an authoritative operational-state context that arrives with the request rather than being inferred from it. Historical traces and retained memory can explain why an accommodation once existed. They should never decide whether it’s still authorized. Memory informs execution; operating mode governs it. And when the two disagree, authoritative runtime state wins.

A small example shows the shape of that context. It carries the minimum needed to bind an exception to a mode, a scope, an authority, an expiry, and a status:

```
{
  "mode": "incident",
  "exception_id": "INC-4721",
  "scope": {
    "segment": "premium-customers",
    "region": "us-east",
    "workflow": "customer-remediation"
  },
  "authority": "service-owner",
  "expires": "incident-close",
  "status": "active"
}
```
The pattern sits near familiar ones without matching them. Feature-flag platforms such as LaunchDarkly target behavior by context, RBAC governs what a principal may do, and tenancy metadata tells a service where a request belongs.3 Operating mode serves a different purpose. It doesn’t replace policy, permissions, or memory: Permissions determine who may act, and policies determine how they may act. Operating mode determines whether exception behavior is authorized at all. As a result, it acts as a higher-order governance constraint on agents, workflows, approvals, tools, and escalation paths. Operating mode draws its authority from the organization’s incident and change process and represents a governed state of the enterprise, consumed by the runtime.

**A design pattern for exception-aware agents**

Operating mode becomes actionable once the platform treats it as a first-class runtime construct. Most agent architectures already inject identity, permissions, tenant context, and policy into every request, and operating mode belongs in that set. Agents consume it as authoritative state after the organization declares the exception, scopes it, assigns authority, and sets an expiry, instead of reconstructing it from prompts or accumulated context.

That reframes the architectural question as whether the platform can guarantee that exception behavior is impossible outside an authorized operating mode. Emergency behavior exists because the platform enables it, and for no other reason. Table 1 shows the minimum contract that makes the boundary testable.

| Field | Purpose | Example | 
| Mode | Current operating state | Normal, incident, recovery | 
| Exception ID | Unique identifier for tracking and validation | INC-4721 | 
| Scope | Boundaries affected by the exception | Workflow, region, customer segment | 
| Authority | Owner who approved the exception | Service-owner | 
| Expiry | When the exception ceases to be valid | Incident closure, timestamp | 
| Status | Current lifecycle state | Active, closed, retired | 

*Table 1. Runtime contract for exception-aware agents*

A workflow invocation receives that state alongside the user request, and orchestration, routing, and tool gateways read the same state. Figure 1 shows the shape.

**Building an exception-aware control plane**

The natural home for operating mode is an external control plane. Incident management platforms, maintenance window services, and change management workflows already hold authoritative operational state, and exception-aware architectures extend those signals into execution with explicit scope, authority, expiry, and closure semantics.4,5 Implementation will vary across organizations, but the principle holds: Exception state should be authoritative, observable, and externally managed, kept out of prompts and workflow definitions and away from agent memory.

**Exception-aware execution**

With explicit operating state, behavior shifts when system state shifts, and prompt wording stops being the lever. Under normal operations, agents run standard workflows, routing, and approvals. Under incident mode, the same workflows can expose scoped accommodations that the authorized mode makes available, such as expedited approvals, alternate routing, deferred reviews, or emergency runbooks.

| Capability | Normal mode | Incident mode | 
| Approval path | Standard workflow | Expedited approval | 
| Queue routing | Primary queue | Alternate queue | 
| Manual review | Mandatory | Deferred where authorized | 
| Tool access | Standard permissions | Emergency tools enabled | 
| SLA handling | Standard policy | Incident response workflow | 

*Table 2. Example runtime behavior across operating modes*

Table 2’s payoff is testability. A workflow in normal mode should never reach an emergency path, and a workflow in incident mode should reach only the accommodations its scope, authority, and expiry allow. Governance becomes an enforceable runtime property the platform can check at execution time.

Return to the opening incident. With operating mode as runtime state, the emergency route opens only inside a declared exception: scoped to the affected segment and workflow, owned by the service owner, and stamped with an expiry tied to incident closure. When the incident closes, the mode returns to normal and the routing gate stops handing that route to the segment, ahead of any manual cleanup. Closure then runs as a check. The platform replays the exception’s scope against live routing, approval, tool, and queue configuration, and confirms that no path still resolves to the emergency behavior. No one has to remember to retire the route; it was bounded by state, and the platform can show it is gone.

**A common operating model for multi-agent systems**

The problem compounds across collaborating agents. Customer-facing, orchestration, and execution agents may share a workflow while disagreeing about state, so one keeps applying emergency routing after another has returned to standard controls. A shared operating state gives them a single governance boundary: The exception is represented once and read consistently everywhere. As agent ecosystems grow more autonomous, shared operational state matters as much as shared identity and authorization. Fragmented state produces fragmented accountability.

**Making exception drift observable**

The payoff is observability. These conditions have historically been hard to see because accommodations scatter across workflow definitions, approval policies, routing configurations, and tool permissions. Explicit operating state makes them measurable at runtime. Closure can trigger automated validation, and the platform can watch for residual exception behavior between closures. The revealing checks are direct: Is the exception’s routing path still reachable, do its temporary approvals or elevated permissions still resolve, and does any expired exception still touch behavior? Tracking how many exceptions are open, how long they stay open, and how often they harden into permanent change turns drift from an audit finding into a monitored signal.

**From adaptive agents to governed systems**

Table 3 captures the shift. Traditional governance documents exceptions and trusts that they are retired. An exception-aware platform represents them as runtime state that can be propagated, validated, monitored, and closed.

| Traditional approach | Exception-aware approach | 
| Exceptions are documented. | Exceptions are represented as runtime state. | 
| Closure is procedural. | Closure is validated. | 
| Audit relies on evidence collection. | Audit relies on observable state. | 
| Exception behavior can persist silently. | Drift becomes detectable. | 
| Agents infer context. | Agents consume authoritative state. | 

*Table 3. Traditional versus exception-aware platforms*

The test is whether the platform can demonstrate that the exception stayed bounded to its purpose and duration.

None of this demands a new governance model. Organizations already know how to govern identity, permissions, change, and operational risk, and the recommendation is to extend those disciplines to operating state. Once operating mode is authoritative, observable, and testable, drift becomes detectable, closure verifiable, and recovery an engineering result. The aim is a platform that can make its own operating conditions explicit, hold every agent to them, and demonstrate a clean return to normal. In that model, temporary accommodations stay temporary, governance lives in the architecture, and enterprise agents earn trust at scale.

## References

- ISO, “ISO 22301:2019: Security and resilience—Business continuity management systems—Requirements,” International Organization for Standardization, 2019. https://www.iso.org/standard/75106.html ↩︎
- Axelos, “ITIL 4: Change enablement,” IT service management practice, 2019. https://www.axelos.com ↩︎
- LaunchDarkly, “Target with Flags,” LaunchDarkly Documentation. https://launchdarkly.com/docs/home/flags/target ↩︎
- ServiceNow, “Change Management,” ServiceNow Product Documentation. https://docs.servicenow.com ↩︎
- PagerDuty, “Maintenance Windows,” PagerDuty Support Documentation. https://support.pagerduty.com/main/docs/maintenance-windows ↩︎
