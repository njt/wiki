# Cloud Agents as OIDC Workloads (Formal)

Formal describes how it eliminated long-lived credentials for cloud coding agents (Cursor, Devin) by treating each agent session as a federated OIDC workload: short-lived tokens, CEL/claim-based policy, and an attested human identity propagated from the agent platform's token so access is scoped by session, repository, and launching engineer. The thesis: agent access is temporary delegation, not another machine secret.

---

## What the source argues

Formal hates secrets — its stated goal is zero long-lived credentials. OIDC federation is the mechanism: a workload presents a short-lived JWT from its own IdP (GitHub env var, cloud metadata endpoint), the relying party verifies it against the JWKS and checks claims (audience-scoped to prevent replay), and Formal turns that into granular authorization via CEL conditions, claim-fed Permissions, SPIFFE JWT-SVID support, and a `CreateFederationToken` endpoint that mints a short-lived Connector credential so workloads never hold static secrets.

The clever part is **human-identity attestation**. Every OIDC integration maps to one machine user, but an integration can name a claim that Formal resolves to the human responsible — a pattern built originally for BI tools like Metabase, where one shared database credential must still be policy-governed per logged-in end user. The trust asymmetry is stated honestly: the machine identity is cryptographically verified; the human identity is only *asserted* by the downstream workload, accepted because the customer already trusts that system. Cloud agent platforms fit this shape perfectly — they can't mint per-human credentials, but they can sign a token saying which session, which repo, which launcher.

Concrete dogfooding: agents request Slack-approved short-lived access to production databases scoped to session ID; a workflow orchestrator without an MCP server is made agent-reachable via a read-only Native User; a Cursor cloud agent federated into a dev Azure account, deployed an AKS cluster from its PR branch, and verified its own Kubernetes-operator change against the Formal dev tenant — closing the loop in a live environment instead of brittle mocks.

## Key quotes

> "We've been reluctant to give these cloud agents access into our infrastructure, as doing so would have meant handing out static credentials without much in the way of granular scoping."

The honest starting point: before OIDC, the choice was agents starved of data or agents holding static production credentials. The engineering contribution is dissolving that dilemma rather than picking a side.

> "Formal can cryptographically authenticate the machine identity, but the human identity is only asserted by the downstream workload. We accept that claim because our customer trusts the system making it."

A rare clear-eyed admission in the agent-identity space: half the identity chain is trust, not proof. The chain is only as strong as the agent platform's honesty about who launched the session — which is exactly the seam an attacker who compromises the platform would exploit.

> "Agent access should be modeled as a temporary delegation — not just as another machine workload with another long-lived secret."

The reusable idea of the piece. "Delegation" implies a delegator, which is what the human-attestation claim supplies; machine-workload framing alone loses the accountability link.

## Key themes

#concept #tool #pattern — OIDC federation and ephemeral credentials as the agent-access mechanism (#concept); Formal's Connector and policy engine as the enforcing tool (#tool); identity propagation from shared-service credentials (Metabase) to agent sessions as a reusable design pattern (#pattern).

## Analysis

This is a vendor post, and it sells — but it sells by dogfooding on real production infrastructure with named, believable workflows, which puts it above most agent-security writing. The strongest contribution is showing that the *hard* part of agent identity isn't the cryptography (workload OIDC is solved, boring even) but the attribution question: who is responsible for this agent run? Formal's answer — reuse the BI-tool end-user-assertion pattern — is elegant precisely because it reuses a trust decision customers already make. Cursor's apparent involvement in shaping the approval flow suggests agent platforms are converging on session-scoped, human-attested tokens as the interop shape.

The weaknesses are the ones the author doesn't dwell on. The human identity is asserted, not proven — the whole model inherits the agent platform's integrity as a trusted base. And the HN comment by anuptalwalkar (2026-10-27) lands a sharper blow: OIDC's trust model assumes the token's subject is the responsible party, and agents break that — "the user can shrug it off as 'the AI went rogue.'" A session-scoped read-only token with Slack approval narrows the blast radius, but it doesn't create a distinct agent role with tightly bounded authority; that remains unbuilt work. Approval-by-Slack is also itself prompt-injection surface: the agent tells the human *why* it needs access, and a compromised agent can social-engineer its own approval. The model is a big improvement over static secrets, not a solution to agent accountability.

## Relations

This source strengthens [[You Dont Want Long-Lived Keys]] with a worked production example of the ephemeral-credentials doctrine applied to agents, not just CI. It nuances [[Identity Management for Agentic AI (OpenID Foundation)]] by showing one concrete implementation of the whitepaper's human-attestation problem. It complicates [[The Agent Access Model]] — Formal's approval-gated session scoping is a partial, shipping instantiation of Cloudflare's proposals, but with the responsibility-attribution gap that model flags still open. And it concretizes [[Zero Trust for AI Agents]]: short-lived tokens and claim-scoped policies are exactly the "hard barriers" Anthropic says survive against agentic attackers.

---
*Sources: [[raw/cloud-agents-oidc-infrastructure]], [[summary/cloud-agents-oidc-infrastructure]]*
*Last updated: 2026-10-28*
