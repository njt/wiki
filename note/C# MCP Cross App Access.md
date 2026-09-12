# C# MCP Cross App Access

Aasawari Sahasrabuddhe's practical walkthrough of building a C# MCP client application secured by Cross App Access (XAA), an emerging open standard that bridges the identity gap between OIDC-authenticated users and downstream MCP servers. In 50 lines of C# and a handful of configuration values, the tutorial demonstrates the full "two-hop token upgrade" — ID token → JWT Authorization Grant → scoped access token — using Okta's `IdentityAssertionGrantProvider` to abstract away RFC 8693 and RFC 7523.

---

## Key Quotes

> "XAA is an open standard that securely enables AI agents to act on behalf of a user" when communicating with downstream services, eliminating repeated manual consent.

The "on behalf of" framing matters. This isn't service-to-service auth where the agent has its own identity — it's delegation. The user logged in, the agent acts with their authority, and XAA makes that chain cryptographically verifiable rather than trust-based. This is the missing piece that makes [[Enterprise-Managed MCP Authorization]]'s zero-touch provisioning actually secure in production.

> "Setting `MapInboundClaims = false` is crucial — it prevents ASP.NET Core from remapping standard claim names, keeping the token payload clean and predictable downstream."

A one-line config flag that would cost you hours of debugging if you missed it. ASP.NET's default claim remapping is well-intentioned (standardizing claim types across providers) but becomes actively harmful in a token-exchange pipeline where claim names must survive intact across two hops. This is the kind of detail that distinguishes a production tutorial from a hello-world.

> "The entire flow — from SSO login to an AI agent fetching data from an MCP server — runs end-to-end in under 50 lines of C#."

The brag is about line count, but the real story is about abstraction quality. The `IdentityAssertionGrantProvider` collapsing two RFCs into a single `GetAccessTokenAsync` call means the SDK authors made the right API design choices. Compare with [[Authenticating MCPs]], where Johnston documents the real-world contortions required when the platform doesn't provide this abstraction layer.

## Key Themes

#mcp #csharp #dotnet #oauth #identity #authentication #xaa #pattern #security #enterprise

## Critical Analysis

**The MCP auth landscape is bifurcating.** On one side: [[Enterprise-Managed MCP Authorization]], where the IdP provisions connectors centrally and users get zero-touch access. On the other: XAA, where the *app* orchestrates token exchange on the user's behalf, and the user's identity flows through the app rather than bypassing it. These aren't competing — they solve different problems. Enterprise-managed auth is for "the user connects their Claude to Salesforce." XAA is for "this custom app needs to talk to an MCP server as the logged-in user." The distinction will matter more as enterprises build their own MCP-native applications rather than just connecting existing SaaS tools.

**The C# MCP SDK is quietly becoming the second most important MCP implementation.** Everyone talks about the TypeScript and Python SDKs because those are what Claude Code and Claude.ai run on. But the C# ecosystem — .NET's enterprise adoption, Okta's investment in builder tooling, and the `IdentityAssertionGrantProvider` shipping auth primitives that don't exist in the other SDKs — suggests C# might become the default choice for *custom* MCP apps in enterprise .NET shops. The xaa.dev playground is the moat-builder: a standardized test environment that makes the first integration trivial.

**What the tutorial doesn't cover is what matters most in production.** Token refresh, error handling when the IdP is down, what happens when the JAG expires mid-session — none of it appears. That's defensible for a "your first XAA app" tutorial, but the jump from 50-line demo to production-ready MCP client is where teams will spend their real engineering time. The `IdentityAssertionGrantProvider` handles the happy path beautifully; the question is whether it also handles the unhappy paths or silently swallows them.

**xaa.dev is a strategic asset masquerading as a playground.** Providing a free, standardized test environment for the XAA flow gives developers a zero-friction on-ramp while also establishing Okta's implementation as the reference. Every developer who builds against xaa.dev is implicitly testing against Okta's interpretation of the spec. This is the Stripe playbook applied to identity standards — and it's smart.

**The C# angle is more significant than it looks.** The MCP ecosystem has been TypeScript/Python-dominant. A first-class C# tutorial from Okta — with an SDK that ships auth primitives the other SDKs lack — signals that MCP is graduating from "AI experiment protocol" to "enterprise integration standard." You don't write .NET tutorials for a protocol that might not stick around.

---

*Sources: [[raw/csharp-mcp-cross-app-access]]*
*Last updated: 2026-07-18*
