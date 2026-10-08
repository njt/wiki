# Securing MCP Servers — Database Access Control

Redgate's Simple Talk guide makes the case that an MCP server wired to a database is a fundamentally different access-control problem than a conventional app: agents decide which tools to call and chain them without per-step human approval, so the "watch the person clicking the button" model breaks. It walks through five failure modes (confused deputy, token passthrough, in-query prompt injection, over-scoped credentials, session hijacking), the MCP spec's OAuth 2.1 authentication model, identity propagation down to row-level security, and a set of ship-by-default hardening choices.

---

## Key quotes

> "AI agents don't ask permission before every query. Instead, they themselves decide which tools to call and chain together. That's a fundamentally different risk model than traditional access control."

The framing that justifies the whole piece. It is the access-control corollary of the loop-agency argument seen elsewhere in the wiki: autonomy is not a permission level you grant, it's the default shape of an agent loop, and security has to be designed for the loop that exists.

> "The user (or its role) should be forwarded all the way down through context to the database, so that row-level security (RLS) and role-based access control (RBAC) can act on real identities."

The structural center of the guide. The common anti-pattern — well-authenticated users, then one wholly-open shared database connection — makes the database blind to who is actually asking. This is the database-layer restatement of the non-human identity argument: agents and tools are principals too.

> "Read-only mode *does* block writes, but it does *not* prevent an agent from returning data that has already been received."

A refreshingly honest admission, and the most important sentence in the piece: the classic "just make it read-only" mitigation does nothing against in-query prompt injection, because exfiltration of readable rows is not a write.

> "The MCP server is not a security firewall. Enforce the same policies at the database."

Policy belongs at the resource, not at the broker. Defense in depth expressed as "the tool layer is convenience, the database is the boundary."

## Themes

#concept #tool #pattern — MCP database security; identity propagation and non-human identity; least privilege at the resource layer; governed change control for agentic writes.

## Analysis

This is a vendor guide (Redgate sells SQL Data Catalog, Flyway Enterprise, and Monitor, and the piece pitches all three), but the vendor-neutral core is sound and unusually concrete for the genre. Its real contribution is the failure-mode taxonomy: the confused deputy and token passthrough sections are accurate to the MCP authorization spec, and the FAQ's blunt answers ("Does read-only mode prevent prompt injection? No.") are the kind of thing practitioners actually need spelled out.

The deepest point is also the one easiest to skim past: identity propagation. Most MCP-in-production deployments today authenticate the human at the client and then let the server hold a single powerful database connection — which means every agent query runs as the server, row-level security never fires, and the audit log says "the app did it." The guide's prescription — per-user connections, RLS deny-by-default, agents as their own principals — is exactly what the [[Least Privilege for AI Agents — Identity, Access, and Tool Binding]] and [[AI Agents Need Their Own Identity and Least-Privilege Access]] arguments look like when they hit an actual schema.

Where the guide is weakest: it treats prompt injection as a residual risk to be contained by access control rather than an attack class in its own right, and its governance story for writes ("route everything through Flyway's pipeline") is a single-vendor answer to a general problem. But its defaults list — ship read-only, validate token audience, refuse passthrough, no session auth — is a genuinely useful checklist.

## Connections

- Strengthens [[Least Privilege for AI Agents — Identity, Access, and Tool Binding]] with a concrete database instantiation: per-user connections, RLS deny-by-default, and per-tool scoping are that argument's access model applied to SQL.
- Complements [[Authenticating MCPs]] by covering the other half of the story — that piece walks the OAuth 2.1 flow itself; this one shows what breaks when the token's audience isn't checked at the database hop.
- Nuances [[Text-to-SQL in the Real World]]: Stonebraker's complaint is accuracy at 10%; Redgate's is that even the accurate queries must run under an identity and row policy — correctness and authorization are separate axes.
- Sharpens [[LLM01 Prompt Injection (OWASP)]] with the specific case of injection payload *stored in database rows* and read back by agents — indirect injection via the data itself, which read-only mode cannot stop.

---
*Sources: [[raw/how-to-secure-mcp-servers-database-access-control]], [[summary/how-to-secure-mcp-servers-database-access-control]]*
*Last updated: 2026-10-08*
