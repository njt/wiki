---
url: https://developer.okta.com/blog/2026/07/16/csharp-mcp-cross-app-access
title: "Build a Secure C# MCP App with Cross App Access (XAA)"
author: Aasawari Sahasrabuddhe
date_fetched: 2026-07-18
date_published: 2026-07-16
---

An Okta Developer Blog tutorial walking through Cross App Access (XAA), an
open standard that lets AI agents act on behalf of a user across downstream
services without repeated manual consent.

XAA works through two RFCs: 8693 (Token Exchange) swaps an OIDC ID token for a
JWT Authorization Grant at the IdP, and 7523 (JWT Bearer Grant) presents that
JAG to an MCP authorization server for a scoped access token. The C# MCP SDK's
`IdentityAssertionGrantProvider` abstracts both into a clean interface.

The tutorial covers wiring up ASP.NET Core OIDC middleware with PKCE,
configuring the two-hop token exchange, connecting an `McpClient` via streamable
HTTP transport with the bearer token, and then calling `ListResourcesAsync` and
`ReadResourceAsync`. The entire flow — SSO login through to fetching MCP server
data — runs in under 50 lines of C#.

[xaa.dev](https://xaa.dev) provides a testing playground for verifying the
end-to-end flow. The full source is at
[GitHub](https://github.com/oktadev/okta-csharp-mcp-sdk-example).
