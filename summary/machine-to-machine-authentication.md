---
url: https://identitysuite.net/blog/identitysuite/machine-to-machine-authentication
title: "Machine-to-machine authentication"
author: IdentitySuite
date_fetched: 2026-09-20
topics:
  - mcp-and-tool-protocols
  - security-and-sandboxing
---

A vendor tutorial from IdentitySuite explaining the OAuth 2.0 Client Credentials flow — the grant type for service-to-service authentication where no human is present. The article contrasts M2M with user-facing OAuth flows: no login screen, no consent dialog, no redirect; the token represents the service itself rather than a person. Because there is no user identity to establish, plain OAuth 2.0 suffices without an OpenID Connect layer.

The walkthrough is end-to-end in .NET: registering the client and scopes in IdentitySuite's admin UI, a `TokenService` that acquires and caches access tokens with a 30-second expiry buffer, attaching the token as a Bearer header on outgoing calls, and validating incoming tokens in the receiving service with OpenIddict introspection — where the claims reflect a service identity (`sub` is the client, plus scopes) rather than a user. A practical aside stresses that service tokens should be cached rather than fetched per call.

The article closes with a decision rule: Client Credentials is for microservices, background jobs, data pipelines, and internal tooling — any flow with no human in the loop — while anything involving a user, even indirectly, belongs in Authorization Code with PKCE. It is introductory material with a product angle, but it states the baseline pattern cleanly and its framing of "service identity" maps directly onto the newer problem of agent identity.
