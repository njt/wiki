# Your Agentic Identity Problem Probably Isn't New

Joe DeCock (Duende Software) makes the standards-first case that "agentic identity" is a marketing label over old OAuth/OIDC questions: agent-for-a-user is an OAuth client, autonomous agent is a workload, and the three genuinely new pressure points — unregistered clients, cross-org authority, credential sprawl — already have new specs (CIMD, ID-JAG, OAuth SPIFFE Client Authentication) built from ideas that sat in niche communities for years until MCP made them mainstream.

---

## Key quotes

> "Those answers do not stop working when the client happens to be an agent. The Model Context Protocol (MCP) reached the same conclusion; its authorization specification doesn't invent a new protocol."

The anti-reinvention thesis in one line. MCP's auth spec is OAuth hardened (PKCE, Resource Indicators, `iss`), not a new design — a useful corrective to the wave of "agents need a new identity layer" pitches.

> "Agents have made these problems mainstream, and the standards community has responded by bringing those niche solutions into OAuth proper."

This is the actual contribution of the piece: the map from agent pain → spec. IndieAuth/Solid-OIDC did CIMD before anyone cared; agent ubiquity is what turned a community hack into an IETF draft.

> "The inconvenience of requiring consent for each step is actually a security risk, because it contributes to consent fatigue; users shown too many prompts are being trained to dismiss them without thinking."

A sharp observation about UX-as-security: bad ergonomics don't just annoy, they actively destroy the human approval gate. Connects directly to consent-fatigue concerns raised in [[Identity Management for Agentic AI (OpenID Foundation)]].

> "The server can't reliably clean up clients because there's no way to tell the difference between a client that is permanently gone, or merely currently inactive."

The cleanest statement of why DCR fails: garbage collection. CIMD sidesteps it by making the client's own domain the source of truth — control of the URL is the registration.

## Key themes

#concept — delegated authority (OAuth for users, workload identity for autonomous agents) as the durable frame; agents don't need a new identity theory
#tool — three specs worth knowing: CIMD (URL-as-client_id), ID-JAG (internal-IdP-brokered cross-org delegation), OAuth SPIFFE Client Authentication (attested workload creds instead of secrets)
#pattern — standards communities mainstreaming niche solutions when a new workload class forces scale (IndieAuth → CIMD)

## Analysis

DeCock's argument is a welcome corrective to vendor-driven "agentic identity" category creation. The taxonomy is honest: most of the problem really is OAuth as usual, and the piece is at its best when it explains *why* each hack failed before the spec arrived — DCR's unbounded client database, domain-wide delegation's all-or-nothing authority, consent fatigue as an attack surface.

Where it's thin: it's a standards-community view that assumes a functioning enterprise IdP and OAuth-literate teams. The SSRF and trust-domain questions CIMD opens get one sentence; the messy deployment reality (agents that *are* the trust boundary violation — see [[AI Agents Need Their Own Identity and Least-Privilege Access]]) is out of scope. And the SPIFFE story presumes infrastructure maturity most orgs running a cron-job agent don't have. Still, as a map of where the standards actually are in late 2026, it's the most concise thing on this subject. The "whose authority is the agent using?" question is the right first question, and it's the same question the OpenID Foundation whitepaper starts from — convergence from a vendor and a foundation is a good sign the framing is stable.

## Related pages

- Strengthens [[Identity Management for Agentic AI (OpenID Foundation)]] — both center the same user-present/user-absent split and consent fatigue; this adds the concrete spec trio (CIMD, ID-JAG, SPIFFE) the whitepaper treats as open work.
- Nuances [[Machine-to-Machine Authentication]] — that tutorial presents Client Credentials as solved and boring; this piece shows where the boring model breaks when clients arrive unregistered and secrets proliferate.
- Complicates [[The Grid — Agent Identity Architecture]] — Galligan's identity-as-configuration for personal agents sits at the opposite end of the trust spectrum from CIMD's domain-controlled federation; together they show "agent identity" means very different things at hobbyist vs. enterprise scale.

---
*Sources: [[raw/agentic-identity-isnt-new-problem]], [[summary/agentic-identity-isnt-new-problem]]*
*Last updated: 2026-10-03*
