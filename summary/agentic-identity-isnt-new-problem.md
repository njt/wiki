---
url: https://duendesoftware.com/blog/agentic-identity-isnt-new-problem
title: "Your Agentic Identity Problem Probably Isn't New"
author: Joe DeCock
date_fetched: 2026-10-03
date_published: 2026
topics:
  - mcp-and-tool-protocols
  - security-and-sandboxing
---

Joe DeCock (Duende Software) argues that "agentic identity" is mostly a relabeling of questions OAuth and OIDC have answered for two decades: who is the user, which client is calling, and what is it allowed to do. Agents acting for a user are just OAuth clients; autonomous agents are just workloads. The MCP authorization spec reached the same conclusion by reusing OAuth hardened with PKCE, Resource Indicators, and the `iss` parameter rather than inventing a new protocol.

What agents genuinely changed is which edge cases became mainstream, and DeCock maps three of them to three new OAuth specifications:

- **Clients nobody registered** — MCP's assumption that every client works with every server broke the register-first model. Dynamic Client Registration turned the auth server's client database into an abuse and maintenance vector, so Client ID Metadata Documents (CIMD) invert the flow: the client publishes metadata at an HTTPS URL it controls, and that URL *is* the client_id. The idea existed for years in IndieAuth, Solid-OIDC, and OpenID Federation; MCP made it load-bearing for everyone.
- **A user's authority crossing trust boundaries** — per-pair consent scatters ungovernable grants; tenant-wide delegation (Google domain-wide, Entra app permissions) hands unpredictable actors too much. The Identity Assertion Authorization Grant (ID-JAG) lets the *internal* IdP exchange the user's ID token for a short-lived assertion bound to the vendor's authorization server, keeping policy enforcement home while the user never sees a consent screen.
- **Manual secret management** — every autonomous agent is a new workload with credentials that get committed, shared, and never rotated. OAuth SPIFFE Client Authentication lets a SPIFFE workload's attested, short-lived identity serve as OAuth client authentication, so the auth server trusts a trust domain instead of holding per-client secrets.

His advice: first decide whose authority the agent uses (user-present → OAuth client; no user → workload identity), use the mechanisms you already have, and focus on the three newly standardized corners.
