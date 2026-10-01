# Identity Management for Agentic AI (OpenID Foundation)

An OpenID Foundation whitepaper (October 2025, lead editor Tobin South, ~20 co-authors spanning Okta, Microsoft, Google, WorkOS and Stanford's Loyal Agents Initiative) mapping the identity, authentication and authorization landscape for AI agents: what OAuth 2.1 + MCP already solves within a single trust domain, and a candid catalogue of what breaks when agents go cross-domain, asynchronous, and recursive.

---

## The Two-Halves Structure

**What works today.** One agent, many tools, one trust domain: OAuth 2.1 with PKCE, externalized authorization (PEP/PDP separation, NIST SP 800-162, being standardized as AuthZEN), SCIM lifecycle management extended with a proposed `AgenticIdentity` schema, CIBA for out-of-band human approval of async actions, and the JWT `act` claim to distinguish the delegating user from the acting agent.

**What doesn't work yet.** Everything past that boundary: cross-domain federation, recursive delegation with scope attenuation, revocation down a chain of offline tokens, consent at agent velocity, browser-level agents that bypass API authorization, and shared agents acting for teams of humans.

## Key Quotes

> "Currently, agents often act indistinguishably from users, creating accountability gaps and security risks."

The paper's central diagnosis, stated in the executive summary. Impersonation is the default posture of today's agents — screen scraping, borrowed credentials, API calls logged as the user. The fix is true on-behalf-of flows where one token carries *two* identities: the user in `sub`, the agent in `act`/`azp`. This is the same first-class-principal argument as [[Least Privilege for AI Agents — Identity, Access, and Tool Binding]] and [[AI Agents Need Their Own Identity and Least-Privilege Access]], but here it comes from the identity-standards establishment rather than the agent-infra world — which is itself the signal: the OAuth ecosystem has decided agents are its problem now.

> "The inability to guarantee timely, system-wide revocation makes proactive risk mitigation essential… credentials can be constrained by execution counts."

The most practically interesting admission in the paper: revocation across delegated chains is "largely unsolved," so the recommended hedge is bounding the blast radius up front — an agent gets N operations, not an open-ended token. This is the identity-layer twin of the blast-radius framing in [[Bounding the Blast Radius — Prompt Injection Defenses]]: when you can't guarantee containment after the fact, cap the damage before it starts.

> "Users will face thousands of authorization requests as agents proliferate, creating security risks from reflexive approval."

Consent fatigue treated as a security vulnerability, not a UX annoyance. The paper's answer is a three-part shift: policy-as-code (an operational envelope, not per-action clicks), intent-based authorization ("book my travel" → a machine-readable least-privilege bundle), and risk-based dynamic authorization (routine actions auto-pass, anomalies trigger CIBA). The diagnosis matches what [[How We Contain Claude]] measured — a 93% approval rate means per-action consent is consent theater at agent speed.

> "An agent wields the delegated authority of a human but operates with the speed and scale of a machine, creating a vastly amplified blast radius for a potential breach."

The reason de-provisioning — not mere token revocation — is elevated to "a foundational pillar of safety": a merely *revoked* agent keeps its registration and trust relationships, a dormant persistent threat. SCIM DELETE plus Shared Signals Framework propagation across all domains, or it isn't dead.

## Key Themes

#concept — impersonation vs. delegation; scope attenuation; consent fatigue; de-provisioning vs. revocation; trust-on-first-contact
#tool — MCP, A2A, CIBA, SCIM AgenticIdentity, SPIFFE/SPIRE, Web Bot Auth, AP2 Mandates, Biscuits/Macaroons, Token Exchange
#pattern — PEP/PDP externalized authorization; two-identity tokens (`sub` + `act`); execution-count-bounded credentials; policy-as-code operational envelopes

## Opinionated Analysis

The paper's strength is honesty about the boundary of the solvable. Section 2 is deliberately boring — "use OAuth 2.1, externalize your PDP, SCIM your agents" — and that boringness is the point: the standards world is telling agent builders to stop reinventing auth badly. The Dynamic Client Registration critique is a good example: MCP's frictionless onboarding created "a large number of anonymous clients" with "a complete lack of a paper trail," which the community has been papering over with Client ID Metadata. The whitepaper names the tradeoff plainly — dev velocity versus accountability — and comes down on the side of accountability for anything enterprise-facing.

Where it is weaker: the future-looking section leans hard on standards that are drafts (OIDC-A, AgenticIdentity schema, Identity Chaining), and its "sovereign agent identity" DID-flavored option reads as a nod to the decentralized-identity constituency rather than a serious near-term path. The browser/computer-use section is the most farsighted and the thinnest — Web Bot Auth as a "passport for agents" raises exactly the two-tier-web problem the paper flags and then sidesteps: who decides which agents are "responsible," and does the anonymous tier become the bot-blocked tier? The multi-user shared-agent problem (the CFO's agent leaking salary data into a channel) is the sharpest unsolved case in the paper, and it gets four sentences.

The connection to the wiki's agent-security corpus is direct but not redundant: this is the *protocol and standards* layer beneath the architecture and threat-model layers. It supplies the mechanism vocabulary — CIBA, Token Exchange, `act` claims, SSF — that [[Zero Trust for AI Agents]]'s hard-barrier principle and [[The Agent Access Model]]'s Trust Ratchet implicitly require but don't specify.

## Relations

- Strengthens [[Least Privilege for AI Agents — Identity, Access, and Tool Binding]]: Microsoft's four pillars get their standards grounding here — dedicated identities via SPIFFE/AgenticIdentity, task-scoped RBAC via intent-based authorization, auditability via the `act` claim.
- Complicates [[The Agent Access Model]]: Cloudflare's multiplayer-access-control admission is confirmed as unsolved by the paper's own multi-user use case, but the paper adds the federation machinery (Token Exchange, identity chaining, Verifiable Credentials) that Cloudflare's proposal leaves open.
- Nuances [[Zero Trust for AI Agents]]: the "hard barriers only" test maps to cryptographic identity and expiring tokens here, but the paper's execution-count credentials and revocation problem show a barrier the Zero Trust doc doesn't treat — authority already granted, propagating offline.
- Extends [[Agentic AI Security Stack]]: Lucktemberg's kill-chain threat model is about attackers; this is about principals. Together they bracket the problem — who the agent *is* (this paper) versus what the adversary *does* to it (the Stack).

---
*Sources: [[raw/identity-management-for-agentic-ai-pdf]], [[summary/identity-management-for-agentic-ai-pdf]]*
*Last updated: 2026-10-02*
