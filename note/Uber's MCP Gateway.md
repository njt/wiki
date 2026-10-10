# Uber's MCP Gateway

Uber's engineering blog on the MCP Gateway: a centralized control-plane/data-plane platform that hosts 800+ MCP servers and 5,000+ tools, auto-generates MCP tools from existing IDL-defined APIs, and solves the context-bloat problem at scale with gradual discovery (Omni MCP), response projection, and a CLI-based Code Mode.

---

## What it argues

Uber's early ad-hoc MCP integrations fragmented fast: hundreds of teams independently building tooling, duplicated infrastructure, tools hard to discover and tightly coupled to specific agents. The Gateway is the fix — a single orchestration and routing layer between agents and back-end services, where AutoCrawler (a Cadence workflow) scans the IDL registry, has an LLM write agent-friendly tool descriptions from Protobuf/Thrift schemas, and registers virtual MCP servers disabled-by-default. The data plane translates MCP calls to HTTP, gRPC, or TChannel through the Muttley service-mesh sidecar, so downstream services change nothing.

The second half is the more interesting one: what breaks at 800 servers. MCP has no native cross-server search, and wiring every server into an agent's context doesn't scale. Uber's three answers — a single Omni MCP proxy with `discover_server`/`discover_tools`/`invoke_tool` for incremental discovery, a GraphQL-like Response Projection that trims tool responses to only the fields the LLM requested, and Code Mode via the `aifx` CLI where agents write tool output to files and grep selectively — are all the same idea applied at different layers: **never load more tool surface than the task needs**.

## Key quotes

> "A core design principle of the MCP Gateway is that discovery doesn't imply exposure. Every MCP server and tool starts in a disabled state and must be explicitly reviewed and enabled by the owning team."

The governance hinge of the whole design. AutoCrawler can register thousands of tools autonomously precisely because nothing is live until a human owner approves it — automation of supply, human control of exposure. Every tool-description change is a config diff requiring owner approval with rollback. This is the pattern that makes auto-generated tools politically survivable inside a large company.

> "If you're building agentic systems at scale, the hardest part isn't the AI. It's building the connective tissue — the discovery, the security, the reliability that makes agents trustworthy enough to act on behalf of real users in a production environment."

The thesis of the piece, and a quietly radical one: the differentiating engineering work has moved below the model. Eight engineers across Uber's Business Platform org wrote this; six are platform/infrastructure people. The AI is assumed.

> "Existing APIs are the fastest way to provide tools to an agent. Rather than asking teams to rewrite their services for an agentic world, MCP Gateway meets them where they are."

The anti-greenfield position. LLM-generated tool descriptions over existing IDLs beat asking thousands of service teams to hand-author MCP servers. It also quietly inverts the usual MCP advice: this is API-mirroring at industrial scale, done successfully — which suggests [[MCPs Aren't APIs — Stop Treating Them Like One]]'s context-economics argument is really about *response* size and tool count per session, not about mirroring per se.

> "Coding agents often operate in shell environments where writing tool output directly to files is more efficient than loading full responses into model context... Code Mode is now the company default for MCP tool use in coding agents."

The most significant admission in the piece: at the company with arguably the largest MCP deployment anywhere, the default path for coding agents is *not* MCP-in-context at all — it's a CLI writing to the filesystem, with grep doing the retrieval. The gateway remains valuable for governance, auth, and redaction, even when its protocol surface is bypassed.

## Key themes

#concept #tool #pattern

## Analysis

This is a strong piece of platform-engineering field reporting, and its two halves disagree with each other in a productive way. The first half is a classic centralization story: fragmented team-by-team integrations replaced by one gateway with registry, ownership, authz, and redaction. The second half is the platform quietly conceding that centralized *protocol* exposure doesn't scale — so it bolts on discovery indirection, field projection, and finally a CLI escape hatch.

The honest reading of Code Mode is that Uber's own agents preferred the training-data-native path (shell, files, grep) over the protocol Uber built. Compare [[The MCP Gateway Iceberg]], which reached the same conclusion from Sierra's much smaller deployment: CLI familiarity beats MCP purity. Two independent gateways, same verdict — that's not a coincidence, it's evidence that tool-definition-in-context is a losing default for coding agents at any real tool count.

The security posture is also more concrete than most MCP writing: tool-level charter policies distinguishing humans, services, and agents; PII redaction out of the box; user-token relay for third-party servers with token exchange at the edge. It pairs naturally with [[Least Privilege for AI Agents — Identity, Access, and Tool Binding]]'s framework, and its "discovery doesn't imply exposure" default is a cleaner enforceable rule than most permission models discussed in the literature.

What the piece doesn't address: evaluation of the LLM-generated tool descriptions (are they good? does anyone measure tool-call success rates?), latency cost of the extra hop, and what happens when an enabled tool's downstream API changes out from under its generated description. Also unexamined is the failure mode of LLM-written descriptions being approved by owners who may not read them carefully — the diff-approval gate is only as strong as the humans behind it.

## Related pages

- [[The MCP Gateway Iceberg]] — Sierra's independent gateway build reaches the same Code-Mode-style conclusion (CLI over MCP purity) at 1/10th the scale; this source strengthens its "CLI familiarity beats protocol purity" claim into a pattern with two data points.
- [[MCPs Aren't APIs — Stop Treating Them Like One]] — complicates it: Uber mirrors thousands of existing APIs as tools and ships fine, suggesting the context-economics problem is solved by projection and discovery indirection rather than by smaller, domain-scoped servers.
- [[Least Privilege for AI Agents — Identity, Access, and Tool Binding]] — Uber's tool-level charter policies and disabled-by-default registration are a production instantiation of that framework's identity-and-binding argument.
- [[Uber — Agentic Engineering Shift]] — places the Gateway inside Uber's broader platform stack; this piece is the deep dive on the layer that post only sketched.

---
*Sources: [[raw/designing-mcp-gateway]], [[summary/designing-mcp-gateway]]*
*Last updated: 2026-10-10*
