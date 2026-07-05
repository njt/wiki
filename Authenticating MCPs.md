# Authenticating MCPs

Matthew Johnston's field guide to three authentication patterns for MCP servers, grounded in real deployments at Jollyes Pets. The framing insight: auth isn't just about security — it unlocks **dynamic tool registration**, the ability to tailor which tools (and even tool *descriptions*) each user sees at runtime.

---

## The Three Patterns

### OAuth 2.0 via SSO

The primary pattern at Jollyes. Claude.ai *can* do OAuth — it just doesn't initiate the flow. When an MCP returns a 401 pointing to `/.well-known/` metadata, Claude auto-discovers the rest. For Entra tenants where Dynamic Client Registration is locked down, pre-register the app and hand Claude a Client ID + Secret.

> "In practice, this has worked fantastically well for the team at Jollyes."

Users SSO into Claude.ai through Entra already, so the MCP piggybacks on that same identity. Non-domain subcontractors are added as guest users rather than building multi-tenant auth — a pragmatic scope constraint.

> Every session a user spawns into Claude triggers a `/validate` call on their MCP, enabling Jollyes to "measure overall AI usage across the business, aside from MCP usage."

This is the sleeper value: auth as observability plumbing.

### No Auth

> "Sometimes we're happy to share!"

Acknowledged and moved on. The honesty is refreshing — not every MCP needs auth, and pretending otherwise leads to over-engineering.

### Token or Hash in the URL

When OAuth is overkill for "quick, short-lived, or light-touch" tools:

```
https://mcp.matthew-johnston.com/mcp?token=XXX
```

Or with verified identity:
```
?user=X&hash=Y  →  sha2(secret_token + user) === hash
```

Johnston openly admits this violates the MCP authorization spec, which prohibits "access tokens in the URI query string." The tension is real: Claude.ai connectors accept *only* a URL — no custom headers, no OAuth initiation UX. When the platform constrains you, you route around the constraint.

> "May you allow MCP setup on Claude.ai with custom headers?"

## Dynamic Tool Registration — the Real Prize

> An API key alone would suffice for security. The real benefit is **dynamic tool registration**.

Since agents request the tool list at runtime, the server returns a *different* set of tools per authenticated user:

- Only the merchandising team sees a write tool for stocking levels
- Tools exist server-side but are invisible to unauthorized users
- Tool **descriptions** can be personalized — `draft_email` includes the user's email address in its description text, so Claude can just "email me the results" without setup

This is elegant. Rather than agents needing to know *about* permissions, they simply can't see what they can't use. The tool catalog becomes a capability boundary, not just a menu.

## The Kaggle Anecdote as Diagnosis

Johnston opens with a failure: Kaggle's MCP in Claude.ai returns 403 on authenticated endpoints. Kaggle documents an `authorize` tool and a `/mcp auth` command (for Gemini CLI), but neither works in Claude.ai's URL-only connector model.

He frames it correctly: "a Claude.ai limitation, not a Kaggle one." His suggestions to Kaggle:

1. Only advertise tools the current auth state permits — don't list everything and fail mid-operation
2. Allow token-as-query-parameter for URL-only clients

Point 1 is the dynamic tool registration argument applied in reverse: if you *can't* do dynamic registration, at least don't pretend all tools are available.

## Critical Analysis

Johnston writes from the trenches — Jollyes Pets is not a hypothetical. The three-pattern taxonomy is sharp because it maps to levels of investment rather than abstract categories: SSO for serious deployments, no auth for public tools, URL tokens as the pragmatic middle when the platform won't cooperate.

The dynamic tool registration insight is the article's real contribution. Most MCP auth discussions stop at "how do I secure this endpoint?" Johnston asks "what does auth *enable*?" and lands on personalization as the answer. The personalized tool description trick — embedding the user's email in a tool's description text so Claude can act on it — is the kind of detail that only comes from actually building and using these systems.

The tension with Claude's MCP spec is honest and productive. Johnston isn't complaining; he's documenting where the spec and reality diverge and asking for the platform to catch up. The URL-header limitation is a real friction point for anyone building MCPs that need to work across Claude.ai (web/mobile) and Claude Code (CLI) — the former constrains the latter in ways that aren't obvious until you hit them.

What's missing: no discussion of token rotation, expiry, or revocation for the URL-param approach. For "short-lived" tokens this is fine, but the pattern is seductive and will be copied for longer-lived use. A footnote on the sharp edges would strengthen the guide.

---

*Sources: [[raw/authenticating-mcps.md]]*
*Last updated: 2026-07-05*
