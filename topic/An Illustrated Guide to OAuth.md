# An Illustrated Guide to OAuth

Aditya Bhargava's visual explainer of OAuth, the authorization delegation protocol that emerged from Twitter in 2007. Walks through the full authorization code flow using YNAB-connects-to-Chase as the running example, then unpacks why every seemingly over-engineered piece exists to close a specific attack vector.

---

## Key Concepts

**The core problem:** how does a third-party app access your data without you handing over your password? OAuth's answer is the **access token** -- an API key scoped to one user with limited permissions.

**The two-stage handshake:** OAuth never puts access tokens in URLs (they'd leak via browser history and server logs). Instead it uses an intermediate authorization code delivered via redirect (front-channel), which the app's backend exchanges for the real token via HTTPS POST (back-channel). This front-channel/back-channel split is the load-bearing design decision.

**The five actors:** Resource Owner (user), OAuth Client (the app), Authorization Server (login + consent), Resource Server (the API), and Scopes (the permissions boundary).

## Key Quotes

> "OAuth's apparent complexity reflects deliberate security design rather than poor specification. Each component addresses specific vulnerabilities discovered through real-world attacks."

> "Never transmit access tokens through URLs."

## How the Flow Works

1. App redirects user to auth server with client ID, redirect URI, requested scopes
2. Auth server validates client ID and redirect URI against registration
3. User logs in and consents to specific scopes
4. Auth server redirects back with an authorization code in the URL
5. App's backend POSTs the authorization code + client secret to get the access token
6. App uses the access token to call the resource server

Every step has a security rationale: redirect URI whitelisting prevents hijacking, client secrets prevent unauthorized code redemption, scopes enforce least privilege, token expiration limits blast radius.

## Variations

- **Implicit flow** -- tokens in redirects, now discouraged
- **PKCE** ("pixie") -- replaces client secret for apps without backends (SPAs, mobile)
- **OpenID Connect** -- identity layer on top of OAuth for "Sign in with" flows
- **Refresh tokens** -- renew access without re-authentication

## Key Themes

- **Layered defence:** each OAuth component closes one specific attack vector #concept
- **Complexity as scar tissue:** the protocol looks over-engineered until you see the exploit each piece prevents
- **Front-channel vs. back-channel:** the fundamental architectural boundary in secure web auth
- **Delegation, not authentication:** OAuth authorizes access to resources; OIDC adds identity on top

## Critical Analysis

This is one of the best OAuth explainers available. Bhargava does what most OAuth docs fail to do: he motivates every design decision by showing the attack it prevents, so the reader understands *why* rather than just memorizing the flow. The YNAB-to-Chase example is well-chosen because personal finance makes the security stakes visceral.

The gap is in the modern reality. PKCE gets a single paragraph, but in 2025 it's the recommended flow for nearly everything -- the "classic" authorization code flow with client secrets is increasingly a server-to-server concern. Mobile and SPA developers reading this might walk away thinking PKCE is a second-class workaround when it's actually the primary path. Similarly, token storage, refresh token rotation, and the OAuth 2.1 draft that deprecates the implicit flow entirely deserve more space.

The piece also doesn't touch the agent dimension at all. In a world where AI agents need to call APIs on behalf of users, OAuth becomes the plumbing that makes [[OneCLI]]-style credential proxies and [[Building Agents for Production Systems with MCP]] viable. The access token is exactly what an agent needs: scoped, time-limited, revocable authority without holding the user's password. [[You Dont Want Long-Lived Keys]] is the natural next step -- once you understand OAuth tokens, the argument for ephemeral credentials writes itself.

Still, as a "what is OAuth and why does it look like this" resource, this is the one to hand someone.

## Cross-Links

- [[You Dont Want Long-Lived Keys]] -- OAuth tokens are the poster child for ephemeral credentials
- [[OneCLI]] -- credential proxy that manages OAuth tokens so agents never touch them
- [[Building Agents for Production Systems with MCP]] -- MCP's OAuth integration for agent-to-service auth
- [[Security and Sandboxing]] -- broader containment context; OAuth is the delegation layer
- [[Designing a Passively Safe API]] -- OAuth as an example of APIs that fail safely by design
- [[Cybersecurity Is Proof of Work Now]] -- the economics of why layered auth matters

---
*Sources: [[summary/an-illustrated-guide-to-oauth]]*
*Last updated: 2026-05-14*
