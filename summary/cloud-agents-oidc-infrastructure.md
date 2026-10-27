---
url: https://www.formal.ai/blog/cloud-agents-oidc-infrastructure/
title: "Cloud Agents, OIDC, and Our Infrastructure"
author: Formal (engineering blog)
date_fetched: 2026-10-28
date_published: 2026 (recent, exact date not stated)
topics:
  - security-and-sandboxing
  - mcp-and-tool-protocols
---

Formal, a data-access control proxy vendor, describes how it uses OIDC federation to give cloud coding agents (Cursor, Devin) auditable, policy-controlled access to production infrastructure without long-lived secrets. The company's stated philosophy is driving long-lived secrets to zero; OIDC lets a workload authenticate with its infrastructure identity instead.

The mechanics: a workload fetches a short-lived JWT from its identity provider (environment variable or metadata endpoint), presents it as a bearer token, and the relying party verifies it against the IdP's JWKS and checks claims — with audience scoping preventing token reuse. Formal layers CEL conditions, claim-based Permissions, SPIFFE JWT-SVID support, and a `CreateFederationToken` endpoint that turns an OIDC token into a short-lived Connector credential so workloads never hold static secrets.

The key design move for agents is human-identity attestation. Each OIDC integration maps to a machine user, but integrations can optionally designate a claim Formal resolves to the human who launched the workload — a pattern originally built for BI tools like Metabase asserting which end user is behind a shared database credential. Cloud agent platforms can't mint per-human credentials, but they can sign tokens naming the session, the repo, and the initiating human, so the same primitive controls agent access by owner's team.

Dogfooding examples: product analytics queries against production databases, where agents request short-lived, approval-gated access over Slack scoped to session ID and database; debugging a workflow orchestrator with no MCP server via a read-only Native User credential; and cloud-native testing where a Cursor agent federated into a dev Azure account to deploy an AKS cluster and verify a Kubernetes operator change end-to-end. The conclusion: agent access should be modeled as temporary delegation, not another machine workload with another secret.
