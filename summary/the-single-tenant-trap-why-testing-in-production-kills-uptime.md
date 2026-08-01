---
url: https://auth0.com/blog/the-single-tenant-trap-why-testing-in-production-kills-uptime/
title: "The Single-Tenant Trap: Why Testing in Production Kills Uptime"
author: Carlos Aguilar — Customer Advocate at Auth0
date_fetched: 2026-07-18
date_published: 2026-07-06
---

An Auth0 blog post arguing that early-stage companies drift into running a single production tenant for speed, then discover the hard way that it cannot be safely partitioned into environments. Workarounds like conditional routing in Actions break down because MFA triggers, global database connections, and session cookies span the whole tenant — any change hits every live user at once.

The article prescribes four fixes. First, create at least development and production tenants, managed as code via the Auth0 Deploy CLI in CI/CD. Second, replace shared dashboard credentials with per-person RBAC, least-privilege on production and permissive on dev. Third, isolate security controls by environment — skip MFA in integration tests but enforce it and breached-password detection in production. Fourth, stream logs to Datadog, CloudWatch, or Splunk instead of relying on the dashboard for observability.

The author contends the migration takes under an afternoon and points readers to Auth0's docs on multi-tenant setup and automated deployment.
