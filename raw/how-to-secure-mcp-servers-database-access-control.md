---
url: https://www.red-gate.com/simple-talk/data-security-privacy-compliance/how-to-secure-mcp-servers-database-access-control/
date_fetched: 2026-10-08
---

**AI agents don’t ask permission before every query. Instead, they themselves decide  which tools to call and chain together. That’s a fundamentally different risk model than traditional access control – and it’s exactly why MCP (Model Context Protocol) servers connected to databases need their own security playbook. **

**This guide covers the failure modes to watch for: confused deputy, token passthrough, prompt injection, over-scoped credentials, and session hijacking. Then, how to  prevent those failure modes – using authentication, authorization, and least-privilege controls.**

## How is MCP database access control different to conventional access control?

**A Model Context Protocol (MCP) server that connects to a database is the link between AI agents and your data. Deploying one requires precise control of who gets which records. **

Conventional access control revolves around monitoring a person. Someone clicks a button, a system performs a permission check, and the action proceeds or fails.

In MCP, on the other hand, an AI agent decides which tools to call, and in what order – *without* human approval of each step. The agent can chain several tool calls and query a production database.

## What can go wrong *without* database access control (the failure modes explained)

Let’s see what failure modes happen *without* access control. Having these in mind allows us to follow access control far more easily.

#### The ‘confused deputy’ problem

The server runs queries with its own privileges instead of the initiator’s. The account *on* the server can read entire tables, so a user who can only see their own rows can request everything. Click here for more.

#### Token passthrough

The server accepts a token and then forwards the unmodified one to the database. The MCP specification on authorization forbids this. The downstream system cannot confirm the token’s intended audience. Passthrough also breaks audience checks, rate limits, and auditing.

#### In-query prompt injection

Read-only mode is not a prevention for in-query prompt injection. Malicious text stored in a row can instruct an agent to return out-of-reach data. The agent may comply even when it only runs read queries.

#### Over-scoped credentials

A single account with deep permissions shared by every user gives every caller deep access.

#### Session hijacking

Performing authentication on a session identifier lets an attacker replay or resume a given session (with the identifier). The security guidance for MCP advises against using session-based authentication.

**More essential reading on Simple Talk**…

Click here for Simple Talk’s full archive of security, privacy and compliance articles and guides.

## How does MCP authentication work?

**Authentication in MCP is transport-dependent. Remote servers over HTTP act as OAuth 2.1 resource servers, and a separate authorization server handles login and issues tokens. MCP servers then validate them.**

The specification requires Proof Key for Code Exchange (PKCE) with SHA-256, HTTPS on every authorization endpoint, and strict redirect URI (Uniform Resource Identifier) validation with two supported discovery mechanisms:

- The server publishes Protected Resource Metadata (RFC 9728) at `/.well-known/oauth-protected-resource`
- The server returns a `WWW-Authenticate`header on a 401 response

This way, a client knows where to authenticate. Clients like Claude Desktop, Cursor, and Claude Code use this method.

### Additional tips

- Always validate the audience. Reject tokens with the `aud`claim that*do not*contain your server, even with signed and/or unexpired tokens.
- Local servers over `stdio`(standard input/output) work differently. Do*not*run the OAuth flow. Read environment variables for credentials.
- Keep the clocks synchronized. Both the MCP server *and*the authorization server should agree on`exp`and`nbf`claims.

## Why you should identity propagation for users, agents, and tools

**The user (or its role) should be forwarded all the way down through context to the database, so that row-level security (RLS) and role-based access control (RBAC) can act on real identities.**

Quite often, the opposite of this is happening. The user is authenticated well but there’s then a wholly-opened and shared database connection for the whole server. The database only “sees” the server and can’t make a difference *between* users.

**Agents and tools are  also principals, so should have their own identities rather than using those of a person. This is called non-human identity (NHI).**

## How to control what each caller can do with authorization

*Authentication* is a process to get a verified identity, whereas *authorization* decides what that identity can do with specific levels.

Scopes define boundaries; just one scope should *not* have both read query and a destructive write. They should be separated.

Additionally, server-level access should be a minimum. Deciding who can connect does not prevent a connected user to call a tool beyond their limits. Scope each user or role to a specific set of tools or operations.

Finally, a per-client consent registry records which applications each user approved, and the server denies requests that don’t match an approval. Start narrow and widen when necessary. This is also known as the principle of least privilege (PoLP).

## Why is it important to enforce least privilege?

**The MCP server is not a security firewall. Enforce the same policies at the database; find out where your sensitive data is  before scoping anything.**

To help with this, you can use data classification tools such as Redgate SQL Data Catalog, which tags any columns containing regulated or personal data.

Apply the following database controls:

- Grant read-only access by default.
- Assign one narrow role per user.
- Combine per-user connections with RLS (row-level security).
- Set RLS (row-level security) to *deny-by-default*.
- Allowlist query types for the tool.
- Send query traffic to a read replica.

To note, none of these controls stop prompt injection on their own. While read-only mode *does* block writes, it does *not* prevent an agent from returning data that has already been received.

## How to govern the write

The control mentioned above is what an agent can *read*, but doesn’t cover what *changes* an agent can make. When an agent can modify schema or data, those changes should be routed through governed change control instead of letting it run ad-hoc SQL.

**Governed change control gives each change its own version, validation step, a record of who (or what) made it, and why it was made.**

Redgate Flyway Enterprise approaches this at the database layer. It applies version-controlled, automated and – most importantly – *deterministic* changes, rather than manual edits.

In addition, the Flyway Enterprise MCP server extends that model to agents, so proposing agentic changes are routed through a governed pipeline used by developers.

That pipeline captures, validates, and records everything. Put simply: compliance is contained *inside* the workflow, rather than being added afterwards.

#### In summary, here’s how you should govern the write:

- Deny direct schema and data changes at the MCP tool layer – and only expose change operations through the governed pipeline. An agent should *not*be permitted to run raw`ALTER TABLE`or`UPDATE`commands against production.
- Connect change control to observability, ensuring that recorded changes line up with the effects on the system. For a wider view, you can connect Flyway Enterprise with Redgate Monitor.

### Future-proof database monitoring with Redgate Monitor

## The defaults you should use to secure MCP database access

**Here are the defaults I recommend to secure your MCP database access, with the aim of giving you as safe a start as possible:**

- Ship read-only; only add write scopes when the workflow requires them.
- Validate the token audience.
- Refuse passthrough.
- Choose a scope naming convention.
- Enable RLS with deny-by-default.
- Do *not*use session-based authentication.
- Keep all server clocks synchronized.

## How to avoid anti-patterns

**Finally, to avoid anti-patterns, you should:**

- Only have *one*shared account with broad rights for every caller.
- Omit the tenant filter in an agent-generated query.
- Assume read-only mode stops prompt injection.
- Authenticate on a session identifier.

## In summary: MCP access control (the key takeaways)

- Authenticate with OAuth for remote servers, and use environment credentials for local servers. Validate token audience on every request. Never forward a client token to the database.
- Authorize per tool – not only per-server. A user who can connect != a user who can delete stuff.
- Apply least privilege at the database. Grant read-only access by default.
- Log the *full*chain of command – including user, agent, tool, query, and row.

**Remember: authentication verifies  who you are, while authorization determines what you’re allowed to access or do.**

### Simple Talk is brought to you by Redgate Software

## FAQs

### 1. What is the "confused deputy" problem in MCP?

It happens when an MCP server runs database queries using its own privileges instead of the requesting user’s. Because the server’s account can often read entire tables, a user who should only see their own rows can end up requesting — and receiving — everything.

### 2. Does read-only mode prevent prompt injection?

No. Read-only mode stops an agent from writing or modifying data, but it doesn’t stop malicious text stored in a database row from instructing the agent to return data it shouldn’t. Preventing this requires access control at the query and row level, not just blocking writes.

### 3. Why shouldn't an MCP server forward a client's token to the database?

This is called token passthrough, and the MCP specification’s authorization guidance forbids it. The downstream system can’t verify who the token was actually intended for, which breaks audience checks, rate limiting, and audit logging.

### 4. How is authentication different for remote vs. local MCP servers?

Remote servers over HTTP act as OAuth 2.1 resource servers, using PKCE, HTTPS, and strict redirect URI validation. Local servers over stdio skip the OAuth flow entirely and instead read credentials from environment variables.

### 5. What does "least privilege" look like in practice for MCP database access?

Grant read-only access by default, assign one narrow role per user, combine per-user connections with row-level security (RLS) set to deny-by-default, allowlist query types, and route production writes through governed change control rather than ad-hoc SQL.

This document contains proprietary information and is protected by copyright law.

Copyright © 2026 Red Gate Software Limited. All rights reserved

Load comments
