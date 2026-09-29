# Designing Agent-First Platforms (Microsoft Foundry)

Microsoft's platform essay argues that the shift from applications that *respond* to agents that *act* breaks the assumptions enterprise infrastructure was built on, and proposes one architecture as the answer: separate the control plane that governs the agent (Foundry, Entra Agent ID) from the isolated runtime where its work executes (Azure Container Apps Sandboxes — per-execution hardware-isolated microVMs with scoped identity, egress control, and pause/resume). Three enterprise case studies (KPMG, Cognite, South Australian Department for Education) all failed on the same axis: not model capability, but somewhere safe and cheap enough to run agent code at scale.

---

## What the argument is

The piece opens with a claim about software architecture history: deterministic code encoded every branch in advance, while multi-agent applications reason out the steps at runtime. That flips the operating model — "The work happens in a loop that no human is standing inside" — and therefore flips what the platform underneath must supply.

The central diagnosis is the **access trade-off** that stalls most agents before production:

> Run the agent on shared infrastructure and every workload inherits the blast radius of every other one. Give it broad access so it can be useful and you have handed an autonomous process far more reach than you intended. Lock it down until it is safe and the agent can no longer do the job you built it for.

The proposed resolution is structural, not procedural: **isolation is built into the runtime rather than wrapped around it**. Each execution gets its own microVM — created in seconds, destroyed after, running as an identity you control, never storing credentials — so teams "stop choosing between a capable agent and a controlled one."

## Key quotes

> That is the job of an agent platform... The Foundry Control Plane governs the agent, but it does not dictate where the agent's work actually runs. That is a separate decision, and a separate layer.

The separation of governance from execution is the essay's one real architectural idea, and it's a good one. It echoes the harness/runtime splits described in [[A Deep Dive on Agent Sandboxes]] and [[Agent Substrate]] — this is now a vendor-consensus pattern, not a research position.

> More than a million sandboxes per day run in production across Microsoft, powering GitHub Copilot, Copilot Studio, Security Copilot, Foundry Agent Service, Azure SRE Agent, and other Azure services.

The strongest evidence in the piece is internal: Microsoft is dogfooding at a scale no customer has reached, and the essay is candid that internal production use is "a sterner test than anything we could design internally."

> The teams who will scale agents successfully over the next few years are making that choice now... treating that separation as a foundational decision rather than something to retrofit once a pilot succeeds.

## Key themes

- #concept — control plane vs. execution plane as the defining agent-platform split
- #tool — Azure Container Apps Sandboxes: per-execution microVM sandboxes with pause/resume and identity-scoped egress
- #pattern — the three case studies all resolve the same access trade-off with per-user/per-task isolation
- #concept — agent identity: agents need their own identity and permission scope, same bar as the rest of the estate ("No one is going to grant an agent an exemption")

## Opinionated take

This is a product marketing essay wearing an architecture essay's clothes, but the architecture is real and the marketing is unusually honest about the failure mode. The "pilot impressed the room, production stalled" framing is accurate and matches what every serious survey of agent adoption reports.

The weak point is the case-study evidence. KPMG, Cognite and EdChat all needed *per-user, per-engagement, per-student* isolation — code-execution sandboxes for interactive product surfaces. That is the easiest sandboxing problem: the workload is short, user-facing, and state-scoped. The harder problem — long-lived autonomous agents acting on shared systems of record with blast-radius consequences — is exactly where the essay waves at pause/resume and moves on. Whether a second-scale microVM startup and ephemeral-credential model survives an agent that needs to hold secrets across a two-day task is not addressed.

The essay also quietly concedes that governance (Foundry) and execution (Container Apps) must *stay coordinated* — "Nothing disappears into the sandbox... what it sent into the sandbox and what came back stay on the record" — which means the two-layer story is really a two-product bundle, and buyers should price the integration work themselves.

Still: the core prescription — dedicated identity, scoped egress, no stored credentials, disposable execution, everything traced — is the right default, and Microsoft's million-sandboxes-a-day claim makes it the most production-tested instantiation of the pattern on the market.

## Related pages

- [[A Deep Dive on Agent Sandboxes]] — this essay is the vendor-scale, productized version of that survey's sandbox taxonomy; its microVM-per-execution design lands firmly in the strong-isolation camp.
- [[Agent Substrate]] — Google's open-source counterpoint to the same problem (gVisor/micro-VM checkpoint/restore on Kubernetes); Microsoft's piece shows the commercial closed twin of the same architecture, pause/resume included.
- [[Building Agents That Don't Break Themselves]] — Botha's "reasoning on durable infra, execution in disposable sandboxes" split is the practitioner-grade articulation of the control-plane/runtime separation Microsoft is selling.
- [[Beyond Zero — Enterprise Security for the AI Era]] — both argue for shrinking trust boundaries to individual agent actions; Microsoft's per-execution sandbox is a concrete enforcement mechanism for Google's floor/ceiling doctrine.

---
*Sources: [[raw/designing-agent-first-platforms-what-changes-when-agents-do-the-work]], [[summary/designing-agent-first-platforms-what-changes-when-agents-do-the-work]]*
*Last updated: 2026-09-29*
