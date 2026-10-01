---
url: https://openid.net/wp-content/uploads/2025/10/Identity-Management-for-Agentic-AI.pdf
title: "Identity Management for Agentic AI"
author: Tobin South et al. (OpenID Foundation, AI Identity Management Community Group, Stanford Loyal Agents Initiative)
date_fetched: 2026-10-02
date_published: 2025-10
topics:
  - security-and-sandboxing
  - mcp-and-tool-protocols
---

An OpenID Foundation whitepaper (lead editor Tobin South, October 2025, with ~20 co-authors from Okta, Microsoft, Google, WorkOS and Stanford) on authentication, authorization and identity for AI agents. It splits into two halves: what works today, and what breaks next.

**Today (Section 2):** OAuth 2.1 with PKCE plus MCP covers the base case — one agent, multiple tools, one trust domain. The recommendations are conventional-but-important: externalize auth decisions to an IdP rather than baking them into MCP servers (PEP/PDP separation per NIST SP 800-162, standardizing via AuthZEN), use SCIM with a proposed AgenticIdentity schema for agent lifecycle management including verifiable de-provisioning, use CIBA for asynchronous out-of-band human approval, and use the JWT `act` claim to separate "who delegated" from "which agent acted" — closing the auditability gap where agent calls are logged indistinguishably from the user's own.

**Tomorrow (Section 3):** The unsolved problems. Agent identity fragmentation (vendors shipping proprietary agent-ID systems); impersonation versus true on-behalf-of delegation (two identities in one token); recursive delegation and scope attenuation (Token Exchange for centralized, Biscuits/Macaroons for offline capability tokens — with revocation across delegated chains "largely unsolved"); consent fatigue at agent velocity (policy-as-code, intent-based authorization, natural-language scopes translated into machine-readable policy); browser/computer-use agents that bypass API authorization entirely (Web Bot Auth as a "passport for agents"); and the economic layer (FAPI for high-stakes APIs, Google's AP2 with Intent/Cart Mandates, KYAPay's Know-Your-Agent cold-start tokens). Six use cases close it out, from consent fatigue through cyber-physical IAM-as-safety-case to the unsolved multi-user shared-agent problem.

The paper's stance is standards-first and anti-walled-garden: reuse OAuth/SCIM/SPIFFE where they work, converge on interoperable profiles (IPSIE), and treat de-provisioning — not just revocation — as the pillar of safety, because a compromised agent "wields the delegated authority of a human but operates with the speed and scale of a machine."
