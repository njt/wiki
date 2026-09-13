---
url: https://www.oreilly.com/radar/operating-mode-as-runtime-state-a-contract-for-enterprise/
title: "Operating mode as runtime state: A contract for enterprise"
author: O'Reilly Radar (no byline shown)
date_fetched: 2026-09-13
date_published: undated (fetched 2026-09-13)
topics:
  - agent-architecture
  - security-and-sandboxing
---

An O'Reilly Radar essay on a failure mode it names **exception drift**: a temporary accommodation granted during an incident — an emergency routing rule, a shortened approval path, an elevated tool permission — outlives the incident and quietly hardens into standard runtime behavior. The opening scenario is a customer-remediation workflow moved onto an emergency route during a service incident; the incident ends, but the route stays active for a customer segment "after everyone has moved on." The route itself was fine and a human approved it; the trouble is that it now runs without a live incident, an owner, or an expiry condition.

The diagnosis is architectural, not procedural. Enterprises already have the human machinery — incident management, change control, postincident reviews — but most agent platforms treat organizational operating state as something outside the runtime rather than an input to it. The runtime knows who is acting and what they may do, but not whether the organization is under normal conditions, incident response, recovery review, or a declared exception. The fix: hand operating mode to agents as **authoritative runtime input**, the way platforms already hand over identity, tenant, environment, and permissions — never inferred from prompts or conversation history.

Agents raise the stakes because they act: they select tools, trigger workflows, and adapt paths at runtime, so an accommodation can spread through routing, tool use, approval paths, and downstream agents at once. The essay's sharpest line separates memory from governance: "Memory informs execution; operating mode governs it. And when the two disagree, authoritative runtime state wins." It proposes a minimum runtime contract — mode, exception ID, scope, authority, expiry, status — served by an external control plane built on incident-management and change-management systems (PagerDuty, ServiceNow), and distinguishes operating mode from feature flags, RBAC, and tenancy: permissions determine who may act, policies how they may act, operating mode whether exception behavior is authorized at all.

The payoff is testability. Under the contract, "a workflow in normal mode should never reach an emergency path," closure becomes a platform-validated replay of the exception's scope against live routing, approval, tool, and queue configuration, and drift — open exception counts, their age, how often they harden into permanent change — becomes a monitored signal rather than an audit finding. A shared operating state across collaborating agents gives multi-agent systems a single governance boundary; the essay closes by arguing no new governance model is needed, only that existing disciplines be extended to operating state.
