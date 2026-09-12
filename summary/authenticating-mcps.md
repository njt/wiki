---
url: https://matthew-johnston.com/authenticating-mcps/
title: "Authenticating MCPs: three ways we do it"
author: Matthew Johnston
date_fetched: 2026-07-05
date_published: 2026-07-01
topics:
  - mcp-and-tool-protocols
  - security-and-sandboxing
---

# Authenticating MCPs: three ways we do it

Matthew Johnston describes three authentication patterns for MCP (Model Context Protocol) servers, drawing from his experience at Jollyes Pets and personal projects. The article opens with a Kaggle MCP authentication failure that surfaces Claude.ai's limitation: connectors accept only a URL with no custom headers or OAuth initiation capability.

## Three Approaches

### OAuth 2.0 via SSO (Primary)
Contrary to the Kaggle experience, Claude.ai *can* do OAuth — it just doesn't initiate the flow. When an MCP returns a 401 pointing to `/.well-known/` metadata, Claude discovers the rest. If the IdP supports Dynamic Client Registration, merely pasting the URL suffices. For Entra (which locks down dynamic registration), they pre-register the app and provide Claude with a Client ID and Secret.

Works "fantastically well" for Jollyes where users already SSO into Claude.ai through Entra. Non-domain users (subcontractors) are added as guest users rather than building multi-tenant auth.

A side benefit: every session triggers a `/validate` call, allowing measurement of "overall AI usage across the business, aside from MCP usage."

### No Auth
"Sometimes we're happy to share!"

### Token/Hash in URL
When OAuth is overkill, they embed identity in query parameters:

`https://mcp.matthew-johnston.com/mcp?token=XXX`

Or for verified identity: `?user=X&hash=Y` where `sha2(secret_token + user) === hash`.

Johnston acknowledges this violates Claude's MCP authorization spec, which prohibits "access tokens in the URI query string." But for "quick, short-lived, or light-touch" tools, the pragmatism wins.

## Dynamic Tool Registration

The real reason authentication matters: **dynamic tool registration**. Since agents request the tool list at runtime, the server returns different tools per user. Only the merchandising team sees a write tool for stocking levels. Even tool *descriptions* can be personalized — a `draft_email` tool includes the user's email address in its description, so Claude can email results without configuration.

## Suggestions for Kaggle
1. Only advertise tools the current auth state permits (don't list everything then return 403)
2. Allow token-as-query-parameter for URL-only clients

## Closing
"May you allow MCP setup on Claude.ai with custom headers?"
