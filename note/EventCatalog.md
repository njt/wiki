# EventCatalog

EventCatalog is a self-hosted documentation tool for software architecture: domains, systems, services, APIs and events as one connected model with schemas, decisions and owners attached, plus an MCP server that lets coding agents query the catalog and an agent layer that keeps docs in sync with code. This is the product homepage, so treat claims as marketing — but the framing of the agent-era problem is unusually sharp.

---

## Key quotes

> "Agents can read your code, but not your architecture. EventCatalog tells them what a change touches, who owns it and why it was built that way."

The cleanest one-line statement of the why-gap that [[Architecture as Code for Agents]] builds a whole essay around. Code answers *what*; ownership, intent and blast radius live outside it. A catalog is one answer to where that context should live.

> "The knowledge exists. It's just scattered. Ownership in spreadsheets. Diagrams in Miro. Schemas in registries. Context in people's heads."

The diagnosis is right and uncomfortable: architecture knowledge exists but has no queryable home, so neither humans nor agents can see the whole system. The remedy — connected resources rather than blank wiki pages — is a plausible middle between Confluence sprawl and full architecture-as-code.

> "Connect Claude, Cursor or any MCP client to the EventCatalog MCP server to query services, schemas and owners as you work."

The MCP move makes the catalog a tool surface rather than reading material — agents pull exactly the service/schema/owner slice they need, scoped per domain. This is the same "context on demand, not context dumped" pattern the wiki keeps seeing in agent-facing infrastructure.

> "EventCatalog Agents update the affected docs as code changes across your organization, and your teams review every update."

The honesty here is the review clause: the docs-stay-current problem is solved by agents writing doc updates and humans approving them, not by trusting generation. That keeps it inside the wiki's verification-first pattern rather than the dark-factory one.

## Themes

- #tool — self-hosted architecture catalog with zoomable domain→service→message views
- #concept — architecture knowledge as an agent-readable context layer ("give them context")
- #pattern — MCP-scoped context provisioning; agent-authored doc updates under human review

## Analysis

The useful claim is the wedge: an agent that can read every file still can't answer "who owns this, and was this boundary intentional?" — which is precisely the knowledge that prevents an agent from confidently breaking a deliberate constraint. EventCatalog's bet is that this context belongs in a structured catalog exposed over MCP, not in prose docs the agent never opens. That positions it as the managed, productized sibling of [[Architecture as Code for Agents]]' FINOS CALM argument: same diagnosis (intent lives outside the code), different mechanism (queryable catalog with ownership and flows vs. machine-readable constraints validated in CI).

What the homepage doesn't say is the hard part: who maintains the catalog. The "Agents update the affected docs" line gestures at it — agents write, humans review — but a catalog that drifts from reality is worse than a Miro board, because agents will *trust* it. The strongest complement in the wiki is [[Archie — An Architecture Agent for Repositories]], which refuses to let observed code promote itself into architecture truth; EventCatalog's agent-sync mechanism is exactly the kind of pipeline Archie's maintainers-rule stance would want to govern. Note also the marketing gloss: 33,000+ catalogs is a vanity metric, and "documentation you can actually trust" is a claim about process, not product — the trust comes from the review gate, not the tool.

For event-driven systems specifically, the connected model (services, messages, flows, owners) addresses the documented pain in [[Change Is Inevitable — Versioning Event-Driven Systems (Bougria)]]: nobody knows which services consume a message, so versioning decisions get made blind.

---
*Sources: [[raw/eventcatalog-dev]], [[summary/eventcatalog-dev]]*
*Last updated: 2026-11-09*
