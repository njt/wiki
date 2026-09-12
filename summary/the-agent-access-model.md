---
url: https://blog.cloudflare.com/the-agent-access-model/
title: The Agent Access Model
author: Cloudflare Research
date_fetched: 2026-08-06
topics:
  - security-and-sandboxing
---

Cloudflare proposes an access control model purpose-built for AI agents, arguing that the controls built for human users (BeyondCorp, SSO, conditional access) fail quietly when applied to agents — granting too much, seeing too little, and trusting for too long. The Agent Access Model (AAM) shifts from trusting the task execution graph to authorizing every individual action against the task and its accumulated state.

Five principles: credentials are short-lived and sender-constrained (RFC 8693 Token Exchange + DPoP binding); enforcement lives in the harness and network, never in the prompt; human oversight is exceptional, not a per-action approval treadmill; grants are reviewed from evidence (the Grant Review Loop proposes narrowing or widening future task templates based on captured activity); and capability state moves in only one direction via the Trust Ratchet — a mechanism that removes capabilities from the task execution graph when protected data is accessed, narrowing the exfiltration window before sensitive data reaches the model.

The architecture has four active controls (Agent Identity Broker, Task-Scoped Access Engine, Mediation Layer across harness and network, Trust Ratchet) and two supporting systems (Agent Activity Log, Grant Review Loop). A worked example — a nightly reconciliation agent — shows how the Trust Ratchet closes external paths before protected financial data enters the model context, making injected instructions harmless.

AAM is honest about its boundary: the single-principal case can be built today with existing standards, but multiplayer access control (agents serving multiple humans with different permissions) remains an open systems problem, with recent research showing privacy-violation rates of 15.8–50.9% in simulated enterprise workflows.
