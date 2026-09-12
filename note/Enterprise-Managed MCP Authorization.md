# Enterprise-Managed MCP Authorization

Anthropic's answer to the MCP enterprise adoption blocker: centralized connector authorization through the identity provider, starting with Okta. Admins provision once, scope by group, manage revocation through the IdP — users get connectors on first login with zero steps. Built on an open MCP extension so any IdP and any connector can plug in. The quiet but important architectural move: making re-auth frictionless so admins can *shorten* token lifetimes.

---

## Key Quotes

> "Now they log in to Claude on day one already connected — 2,000 employees, provisioned through Okta, zero extra steps."

— Cameron Leavenworth, Ramp. This is the before/after in one sentence. Previously every user had to auth every connector individually. Now: nothing.

> "The momentum around MCP is incredible, but as we move toward an interconnected AI workforce, security can't be an afterthought."

— Aaron Parecki, Okta. The subtext: MCP was built for developer enthusiasm first, enterprise security second. This is the correction. Parecki is the right person to deliver it — he's Okta's Director of Identity Standards, not a product marketer.

> "Enterprise-managed auth is a foundational milestone in realizing Asana's vision as the operating system for human-agent teams."

— Arnab Bose, Asana CPO. Ambitious framing but not wrong. If MCP connectors are how agents reach into enterprise tools, centralized auth is the difference between "works for the pilot team" and "deployable to the whole company."

> "The only way to use Supabase through Claude was to be an org owner or hand out Personal Access Tokens..."

— Bil Harmer, Supabase CISO. The most honest quote in the piece. Before enterprise-managed auth, the options were maximum privilege or credential sharing. Neither scales past a team of five.

## Key Themes

#mcp #security #enterprise #identity #authorization #pattern #claude

## Critical Analysis

**The open extension is the right call.** Anthropic could've built something Claude-specific that locked in the ecosystem. Instead they extended the MCP spec with an open authorization extension that any IdP (not just Okta) and any connector (including custom ones) can implement. This is the playbook from [[Building Agents for Production Systems with MCP]] — MCP as the compounding layer — applied to the auth problem specifically.

**The real innovation is invisible.** The obvious win is "zero-touch setup." The subtler one: because re-auth through the IdP is instant and frictionless, admins can now *shorten* access token lifetimes dramatically. When auth costs the user nothing, you can afford to check more often. This inverts the traditional security/UX tradeoff — better security *improves* UX rather than degrading it.

**What's missing:** The announcement is heavy on launch partners and light on what happens when the IdP is unavailable. If connector access depends on an Okta check, does that become a hard dependency for Claude chat and Claude Code to function? The article doesn't say. Also unmentioned: how this interacts with [[How We Contain Claude]]'s finding that custom code around proven primitives is always the failure point. The extension is open — which means implementations will vary in quality.

**The Supabase quote is a Rorschach test.** Bil Harmer's admission that the previous state was "be org owner or share PATs" tells you the actual enterprise readiness of MCP before this launch. The partner quotes are PR-vetted, but that one slipped through with real honesty. MCP has been a developer protocol; enterprise-managed auth is the start of it becoming an enterprise protocol.

**The Slack mention is the most interesting.**
> "Slack is the place where humans and agents are working side by side, in the same conversation..."

Rod García's framing suggests Slack sees MCP connectors not as a dev tool integration but as a *collaboration primitive*. If agents participate in Slack conversations through the same auth pathway as humans, the boundary between "bot" and "coworker" gets blurry fast. This connects to [[Agent Identity]] — auth isn't just about permissions, it's about establishing *who* the agent is in the organizational graph.

## Related Pages

- [[Building Agents for Production Systems with MCP]] — MCP as the standard integration layer
- [[Control Plane MCP Server]] — most complete vendor MCP implementation, with guardrail layer
- [[An Illustrated Guide to OAuth]] — the auth delegation patterns this builds on
- [[How We Contain Claude]] — security thinking at Anthropic; custom code as failure point
- [[Security and Sandboxing]] — hub page for agent security
- [[Agentcookie]] — session/auth state management for agents
- [[AI Engineering for Developers]] — covers MCP/A2A in the agent stack
- [[Running an AI-Native Engineering Org]] — the Claude Code team's practices
- [[Agent Identity]] — auth as identity, not just permissions
- [[10 Principles for Agent-Native CLIs]] — designing for agent consumers

---
*Sources: [[summary/enterprise-managed-auth]]*
*Last updated: 2026-06-21*
