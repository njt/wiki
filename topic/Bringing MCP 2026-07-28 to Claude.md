# Bringing MCP 2026-07-28 to Claude

Anthropic ships the fifth MCP spec release, anchored on three architectural bets: a stateless request/response core (goodbye bidirectional sessions), standardized extensions for Apps and Tasks, and production-grade OAuth 2.0/OIDC alignment. The ecosystem quotes — from Figma, Intuit, Netlify, PostHog, Xero, and Zoom — read like a roll call of companies voting with production deployments on the new spec during beta. Claude's own MCP surface now spans 950+ published servers, enterprise-managed auth via Okta/Entra, developer observability dashboards, and a research-preview tunnel for private-network connectors.

---

## Key Quotes

> "The stateless core in the 2026-07-28 spec makes MCP a first-class HTTP workload with no session management to work around."

— Sean Roberts, Netlify VP of Applied AI. This is the headline in one sentence. The old bidirectional stateful model was fine for local stdio connections but became a deployment tax the moment you tried to put an MCP server on the internet. Stateless HTTP changes the infrastructure calculus entirely: serverless, edge, CDN — the whole modern deployment surface opens up.

> "Admins authorize a connector once, users inherit access through their existing IdP groups."

— From the enterprise-managed auth announcement. This collapses what was previously an N×M matrix of per-user, per-connector OAuth handshakes into a single provisioning action. The architectural insight is that the IdP group is the right abstraction, not the individual user. [[Enterprise-Managed MCP Authorization]] goes deeper on the Okta integration.

> "MCP is currently the right tool for orgs and enterprises."

— Not from this article, but from Charles Chen's [[MCP Is Dead; Long Live MCP]], which pairs well here. The 2026-07-28 spec addresses Chen's core complaint — that MCP over stdio was unnecessary complexity for solo devs — by making MCP over HTTP the default deployment model. Stateless HTTP + production OAuth is the architecture that justifies Chen's enterprise thesis.

> MCP surpassed 400M monthly SDK downloads — a 4x increase this year.

The number that makes the spec transition matter. This isn't a protocol in search of adoption; it's a protocol with enough gravity that breaking changes (and stateless core *is* a breaking change) require real coordination. The ecosystem quotes are the evidence that coordination happened.

## Key Themes

#mcp #protocol #specification #enterprise #oauth #stateless #claude #anthropic

## Critical Analysis

**The stateless pivot is the right call, a year later than it should have been.** Bidirectional stateful protocols are a deployment tax. Every infrastructure team that tried to put an MCP server behind a load balancer learned this the hard way. The move to request/response — essentially making MCP a well-structured HTTP API with a standard discovery mechanism — eliminates the session-affinity problem that made MCP servers annoying to operate. Netlify's quote is telling: they wanted MCPs to be "as simple as the rest of the platform," and they couldn't say that before.

**The extensions framework is the sleeper.** Everyone is focused on stateless (because it's the breaking change) and auth (because it's the enterprise blocker), but versioned extensions are the architectural move that prevents MCP from collapsing under its own success. Without a formal extension mechanism, every new capability — interactive UIs, long-running tasks, streaming — would require a core protocol change. The extension framework means MCP Apps and MCP Tasks can evolve on their own cadence without destabilizing the tool-calling layer everyone depends on.

**The auth story is finally honest about enterprise reality.** The old MCP auth model assumed developer enthusiasm would carry the day. It worked for hackathon demos and solo projects. It didn't work for companies with Okta or Entra tenants, security review boards, and compliance requirements. Aligning with production OAuth 2.0 and OIDC deployments isn't innovative — it's catching up to where every other enterprise protocol already was. But it's catching up that matters; the gap was the adoption ceiling. [[Authenticating MCPs]] documents the patterns developers used to work around the gap.

**The MCP tunnels preview is more strategic than it looks.** The hardest MCP adoption problem isn't building servers — it's connecting Claude to servers that live inside a corporate network behind a firewall. If the answer is "expose it to the public internet," most enterprises will say no. Tunnels solve the deployment problem without the security argument. This is the feature that turns MCP from "works on my machine" to "works in my company." It's research preview for now, but if Anthropic ships it, it's the most important Claude-specific feature in the announcement.

**The spec change is validated by independent builders shipping against it.** Simon Willison built three tools — mcp-explorer, datasette-mcp, and llm-mcp-client — against the new stateless spec in a single week. His datasette-mcp plugin had failed three times under the old stateful protocol; the stateless redesign removed the complexity that blocked it. When a spec change turns a four-attempt failure into a one-week success, it's not just cleaner on paper — it's genuinely more buildable. See [[Stateless MCP]] for his full account.

**What's missing: migration tooling and a deprecation timeline.** The article says "support is rolling out across Claude products soon" but doesn't address what happens to the 950+ existing MCP servers built against the old stateful protocol. A spec transition of this magnitude needs a migration guide, compatibility window, or at least a deprecation schedule. The ecosystem quotes suggest key partners already built against the new spec during beta, but that's the top of the pyramid — what about the long tail of community servers?

**The observability dashboard is a product play disguised as developer tooling.** Giving published connector authors performance data isn't altruism — it's the infrastructure for an MCP marketplace. Adoption metrics, latency analysis, usage breakdowns by product: these are the analytics you build when you're planning to let developers charge for connectors. The observability feature is also the quality flywheel: bad connectors get diagnosed and fixed or abandoned, good ones get visibility, and the ecosystem improves without Anthropic needing to curate it manually.

---

*Sources: [[raw/bringing-mcp-2026-07-28-to-claude]]*
*Last updated: 2026-08-01*
