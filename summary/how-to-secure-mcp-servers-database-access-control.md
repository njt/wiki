---
url: https://www.red-gate.com/simple-talk/data-security-privacy-compliance/how-to-secure-mcp-servers-database-access-control/
title: "How to Secure MCP Servers: Database Access Control"
author: Redgate Software (Simple Talk)
date_fetched: 2026-10-08
date_published: 2026
topics:
  - security-and-sandboxing
  - mcp-and-tool-protocols
---

A practitioner's guide from Redgate's Simple Talk on why MCP servers connected to databases need their own security playbook. The core argument: an agent decides which tools to call and chains them without human approval per step, so the conventional "monitor the person clicking the button" access-control model no longer applies.

The guide catalogs five failure modes — the confused deputy (the server runs queries with its own privileges instead of the initiator's), token passthrough (forbidden by the MCP authorization spec because it breaks audience checks, rate limits, and auditing), in-query prompt injection (malicious text stored in a row can steer an agent even under read-only mode), over-scoped shared credentials, and session hijacking. Prevention follows the MCP spec's OAuth 2.1 model: remote servers act as OAuth resource servers with PKCE, HTTPS, and strict redirect validation, discovering authorization via Protected Resource Metadata or `WWW-Authenticate`; local stdio servers skip OAuth and read credentials from environment variables.

The strongest structural recommendation is identity propagation: forward the user's identity all the way down to the database so row-level security and RBAC act on real identities, and give agents and tools their own non-human identities rather than borrowing a person's. Authorization should be scoped per tool, not just per server, with a consent registry and least privilege by default. At the database layer the recommended defaults are read-only access, one narrow role per user, per-user connections, RLS deny-by-default, allowlisted query types, and read replicas. Writes should route through governed change control (Redgate pitches Flyway Enterprise here) rather than ad-hoc `ALTER TABLE` or `UPDATE` from an agent. The guide closes with a candid note that none of the read-side controls stop prompt injection on their own — read-only mode blocks writes but not the exfiltration of already-reachable data.
