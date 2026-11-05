---
url: https://www.nist.gov/blogs/cybersecurity-insights/design-tenets-agent-identity-management-systems
title: "Design Tenets for Agent Identity Management Systems"
author: NIST Cybersecurity Insights (NCCoE)
date_fetched: 2026-11-06
date_published: ~2026-10/11 (post #2 in an ongoing series)
topics:
  - security-and-sandboxing
  - agent-architecture
---

# Design Tenets for Agent Identity Management Systems

The second post in NIST's series on agentic identity, authentication and authorization proposes ten "Design Tenets" for IAM systems that manage agents across enterprise and consumer use cases. The tenets are drawn from public comments on the NCCoE agent identity and authorization project and stakeholder engagement.

## The ten tenets

1. **Distinct identity** — agents get unique, cryptographically verifiable identities separate from humans and service accounts, tied to workload and configuration; distinct lifecycle from human principals.
2. **Chain of trust with attenuation** — agents are cryptographically linked to the initiating entity; downstream and sub-agent scopes attenuate so no downstream entity can claim more authority than its parent.
3. **Zero standing privilege** — no persistent rights when idle; least privilege plus separation of duties.
4. **Just-in-time access with runtime verification** — every tool invocation is re-verified against current state and risk; no long-lived bearer tokens or static secrets.
5. **Human escalation on risk thresholds** — defined tolerances trigger human review with enough context to decide; explicitly warns against **consent fatigue**.
6. **Cryptographically signed mandates** — task originators sign statements of intended outcomes and constraints, verifiable downstream for authorization, auditing, and non-repudiation.
7. **Architectural separation of policy from reasoning** — the logic that decides what an agent tries to do is separated from what it is allowed to do; "guardrails alone are insufficient" because LLMs see data and instruction as one token stream.
8. **Deterministic access decisions** — probabilistic signals may inform access decisions but deterministic mechanisms must be the primary enforcer.
9. **Agent-specific monitoring** — detection tuned to agent scale and velocity, with metrics, thresholds, and policies of their own.
10. **Independent, tamper-evident, append-only audit** — an evidence chain of identity, delegated intent, and actions that the agent itself cannot modify.

NIST stresses the tenets compose: preventing a "confused deputy" attack needs #2 plus #3, #7, and #8 together, not any one alone. They admit current standards don't yet fulfill all ten, framing them as an assessment baseline and roadmap, with NCCoE demonstrations to follow.

## The comments are part of the value

The post draws substantive public comments: a founder of newauth argues the chain of trust ends at a device, not a person, since passkeys live in the same browser as the agent — proposing that high-impact approvals require a fresh human response nothing in the agent's environment can generate. A financial-services reader asks what happens when mandate issuer, agent operator, and enforcing party are different organisations (a consumer's agent asking a bank to pay). A veteran IAM engineer answers with the need for a neutral, vendor-independent registry anchoring mandate chains, and for revocation receipts independent of the agent's cooperation. Another commenter maps tenet #2 to OWASP AISVS 9.5.7 and notes no tenet covers reversibility classification of high-impact actions (AISVS 9.2.3/9.2.4).

---
*Sources: [[raw/design-tenets-agent-identity-management-systems]], [[summary/design-tenets-agent-identity-management-systems]]*
*Last updated: 2026-11-06*
