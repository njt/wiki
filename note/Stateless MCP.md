# Stateless MCP

Simon Willison's return to MCP after the 2026-07-28 stateless spec: the architectural simplification from two-request sessions to single HTTP calls, three tools built in a week as validation, and the security argument that explicitly-declared MCP tools are easier to audit and control than granting agents arbitrary shell access.

---

## Key Quotes

> "Giving an agent a shell environment with the ability to access the internet is fraught with risk, and requires a strong model that is capable of effectively driving such an environment. MCP tools are easier to audit and control, and simple enough that smaller models that run on a laptop can still drive them reasonably well."

The core security thesis in two sentences. Willison is pushing back against the dominant pattern in coding agents — drop the model into a shell with `curl` and let it figure things out. His argument is not that MCP is perfectly safe (he flagged prompt injection risks in 2024), but that the threat surface is *legible*: you can audit the tool list, restrict what each tool can do, and reason about blast radius. Arbitrary shell access is illegible by design.

> "This is probably the fourth time I've tried building this plugin, but thanks to the new stateless MCP specification I finally have a version that feels good to release."

The most telling line in the piece. Willison is one of the most prolific and capable open-source developers working with LLMs. If he tried four times to build an MCP Datasette plugin and only succeeded when the spec simplified, that's not a skill issue — it's an architecture verdict. The old stateful MCP imposed real implementation tax that stateless MCP removes.

> "I plan to lean into MCP a whole lot more when I'm building sensitive applications on top of LLMs."

This is the directional signal. Willison has been writing about LLMs since 2022, building practical tools the whole time. When someone with his track record of shipping announces a strategic pivot toward a technology, it's worth paying attention. The word "sensitive" does the work here — MCP is his answer to the trust problem.

> "I find building CLI tools like this to be a really productive way to get familiar with a specification, even if an agent writes most of the actual code."

A meta-observation about Willison's method. He uses coding agents (Codex, specifically) to build tools against a spec as a way of learning the spec. The agent produces the code; he produces the understanding. This is the [[Understand to Participate]] thesis in practice — building to learn, not just to ship.

## Key Themes

#mcp #protocol #stateless #security #tool #simon-willison

- **#concept Stateless MCP**: The 2026-07-28 spec eliminates the initialize-session handshake. One HTTP request carries the method, tool name, and parameters as headers. No session affinity, no server-side state. MCP becomes a deploy-anywhere HTTP workload — serverless, edge, CDN.
- **#concept MCP as safety boundary**: Explicitly declared tools are auditable; arbitrary shell access is not. The Lethal Trifecta (tools + internet + automation) is harder to contain when the agent's toolkit is "anything a shell can do." MCP constrains the toolkit to named, inspectable operations.
- **#tool mcp-explorer**: Stateless Python CLI for interactively probing MCP servers — `list`, `inspect`, and `call` subcommands. Runs via `uvx` with no install. The 2026 equivalent of `curl` for APIs: a universal client for poking at MCP endpoints.
- **#tool datasette-mcp**: Datasette plugin exposing a `/-/mcp` endpoint with three tools: `list_databases()`, `get_database_schema()`, `execute_sql()`. Read-only SQL for now. Fourth attempt, first success — stateless MCP removed the complexity that blocked earlier versions.
- **#tool llm-mcp-client**: Alpha plugin for Willison's LLM CLI. Invoke MCP servers inline with `MCP("url")` syntax in prompts. Under consideration for LLM core once baked.

## Critical Analysis

**The stateless redesign is the spec actually earning its keep.** The old MCP was designed for local stdio connections — a single developer's machine, one client, one server. That model broke the moment anyone tried to put an MCP server on the internet. Session state meant load balancer affinity, sticky sessions, and stateful infrastructure — everything that makes web services hard to operate. Stateless MCP fixes this by making the protocol honest about what it is: HTTP request/response with structured discovery. [[Bringing MCP 2026-07-28 to Claude]] covers Anthropic's official framing of this transition; Willison's contribution is the practitioner's validation — three tools built in a week, two of them useful enough to release.

**The security argument is correct but incomplete.** Willison is right that MCP tools are easier to audit than arbitrary shell access. But the comparison is MCP vs. *current* coding agents, not MCP vs. a well-designed alternative. A coding agent with a carefully scoped set of CLI tools — a whitelist of commands, not a shell — could achieve much the same auditability as MCP while retaining the flexibility of local execution. The real security win of MCP is not the protocol but the *mindset* it imposes: declare what the agent can do, don't just give it access and hope. [[The MCP Gateway Iceberg]] shows this mindset operating at enterprise scale, with multi-pass data guards and dual identity models that push far beyond what individual developers implement.

**Willison's method is the hidden story.** He uses Codex to build tools against a spec he's learning, validating the spec through implementation. The tools are real — mcp-explorer is genuinely useful — but they're also a learning exercise. This is the compound engineering pattern from [[Agent Coding Workflow]]: build the harness (the CLI tool), use the harness to understand the system, then build the real thing (datasette-mcp, llm-mcp-client). The sequence matters. He didn't start with the Datasette plugin; he started with the explorer, learned what the spec actually does versus what it claims, and only then built the integration.

**The "simpler for smaller models" point deserves more attention.** Willison notes in passing that MCP tools are "simple enough that smaller models that run on a laptop can still drive them reasonably well." This is a deployment argument hiding in a UX observation. If MCP tools work with local models on consumer hardware, then MCP isn't just an Anthropic platform play — it's infrastructure that works at every scale from laptop to datacenter. That's the difference between a protocol that's useful for one company's products and a protocol that's useful for the ecosystem.

**What's missing: the migration story for existing MCP servers.** Willison doesn't address what happens to servers built against the old stateful spec — he's building new things against the new spec and doesn't need to care. But the broader ecosystem does. The [[Bringing MCP 2026-07-28 to Claude]] page notes the 950+ published servers that exist; every one of them has a migration decision to make. Willison's experience — that stateless MCP made his fourth attempt succeed where three failed — suggests the migration is worth it. But he doesn't tell you how to do it.

**The llm-mcp-client is the most strategic of the three.** mcp-explorer is a learning tool; datasette-mcp is a specific integration. But llm-mcp-client — embedding MCP tool access inside a general-purpose LLM CLI — is the pattern that generalizes. If Willison brings it into LLM core, every LLM user gets MCP access as a built-in capability. That's the path from "protocol with 400M SDK downloads" to "protocol that's invisible infrastructure, like HTTP." [[Building Agents for Production Systems with MCP]] makes the case for MCP as the standard agent-to-production integration layer; llm-mcp-client is what that looks like at the command line.

---

*Sources: [[raw/stateless-mcp]], [[summary/stateless-mcp]]*
*Last updated: 2026-08-06*
