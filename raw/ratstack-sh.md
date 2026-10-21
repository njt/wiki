---
url: https://ratstack.sh/
date_fetched: 2026-10-21
---

# Rat Stack

> For agents: start with the [agent guide](https://ratstack.sh/llms.txt). Every page is Markdown by default; add `Accept: text/html` for HTML.

Free workshop: [Build Agent Loops That Never Stop Shipping](/tokenmaxx#interested).

Most of this site is for you to hand to your agent and use in your own setup. A lot is generated, and some of it reads weird. I'm rewriting it as I go, so it's better to read for me and for you. Comments, feedback, and discussion are welcome: [@joelhooks on X](https://x.com/joelhooks).

Build an app and its cloud as one typed program: TypeScript checks how the parts fit together. [Effect](https://effect.website/docs/getting-started/introduction/) is a TypeScript library for dependencies, errors, and concurrent work. [Alchemy](https://ratstack.sh/skills/learn-alchemy) declares cloud resources in TypeScript and infers their connections from the code. Our goal is the best Effect + Alchemy application we can build.

*An Effect stack so pure [(aspirational)](/debt.md) Kit Langton will blush.*

The [fence](https://ratstack.sh/lore/the-fence) is our compiler checks, lint rules, and commit hooks. They reject prohibited code and shortcuts.

Copy rat-stack into your project and keep its source under your control. Keep the packages you need and follow the removal checklists for the rest.

## Connect an agent

Paste this into your coding agent:

```text
Read https://ratstack.sh/llms.txt and use rat-stack as the reference
for how we build: Effect for the hard parts, Alchemy for the
infrastructure, and a fence that makes the easy path the right
one. Search its rules and skills before you write code, follow
its patterns, and tell me when my code breaks them.
```

```text
Read https://ratstack.sh/llms.txt and use rat-stack as the reference
for how we build: Effect for the hard parts, Alchemy for the
infrastructure, and a fence that makes the easy path the right
one. Search its rules and skills before you write code, follow
its patterns, and tell me when my code breaks them.
```

Or connect the [MCP (Model Context Protocol)](https://ratstack.sh/lore/mcp-is-another-surface) server to let your agent call the site's tools:

```sh
# Claude Code
claude mcp add --transport http rat-stack https://ratstack.sh/mcp

# Codex
codex mcp add rat-stack --url https://ratstack.sh/mcp
```

```text
# Claude Code
claude mcp add --transport http rat-stack https://ratstack.sh/mcp

# Codex
codex mcp add rat-stack --url https://ratstack.sh/mcp
```

Cursor reads `~/.cursor/mcp.json`:

```json
{ "mcpServers": { "rat-stack": { "url": "https://ratstack.sh/mcp" } } }
```

```text
{ "mcpServers": { "rat-stack": { "url": "https://ratstack.sh/mcp" } } }
```

Load a skill into one session when the task needs it. Skills stay out of global and project skill folders. The [skills index](https://ratstack.sh/.well-known/agent-skills/index.json) lists every skill and its `SKILL.md` URL.

```text
Read https://ratstack.sh/.well-known/agent-skills/index.json. Pick the skill that fits my task, fetch its SKILL.md, and follow it in this session only. Do not install it or write it to any skills folder.
```

Supports MCP protocol versions 2026-07-28, 2025-11-25, 2025-06-18, 2025-03-26, and 2024-11-05. Clients on protocol 2026-07-28 use no sessions. Older clients get their own session in a Cloudflare Durable Object, which stores state between requests.

- [MCP connection details](https://ratstack.sh/.well-known/mcp.json)
- [HTTP API docs](https://ratstack.sh/openapi.json)
- [Short agent guide](https://ratstack.sh/llms.txt)
- [Public rules, lore, systems, and skills](https://ratstack.sh/llms-full.txt)
- [Lore wiki](https://ratstack.sh/lore)
- [Systems](https://ratstack.sh/systems)
- [CAFE news](https://ratstack.sh/news) and the [CAFE directory](https://ratstack.sh/directory): reviewed Cloudflare, Alchemy, Foldkit and Effect news and projects

## Four ideas

- **Pieces:** A service is an interface for one job. A [Layer](https://ratstack.sh/lore/layer-constructor-pattern) describes how to build its implementation and supply its dependencies. An Alchemy Layer also carries its cloud resources. Choose the implementation when you provide the Layer.
- **Trust.** Make the easy path the right path. The codebase and the compiler stop mistakes that rules and style guides can only ask about. Remove a cloud resource connection and the code that uses it stops compiling.
- **Reliability.** Reduce mistakes so an agent can work longer without help.
- **More useful features.** Build more of the jobs your product needs. Layers can carry cloud resources alongside application code. The compiler checks resource requirements; deployment checks prove that the result works.

See the [vision](https://ratstack.sh/VISION.md) for the reasoning and sources.

## Packages and their jobs

rat-stack ships these packages and apps. Remove the ones you don't need with the [removal checklists](https://ratstack.sh/README.md#keep-or-cut). Trace their imports and run the checks after each cut.

| Package or app      | Job                                                                 | Removal                                  |
| ------------------- | ------------------------------------------------------------------- | ---------------------------------------- |
| capability          | Shared data definitions and command-line, web, and agent interfaces | Keep the interfaces you use              |
| core                | Shared operations and service interfaces                            | Keep what your app uses                  |
| database            | Run logs with D1 or Hyperdrive Postgres                             | Remove dependent services too            |
| auth                | Identity and access through Better Auth                             | Remove dependent services too            |
| devtools            | Record, replay, and compare operation calls                         | Optional development tools               |
| events              | Capture request analytics                                           | Optional                                 |
| lore                | Query the public content graph                                      | Remove dependent content operations too  |
| subscriber-delivery | Handle signup and email confirmation                                | Remove dependent signup operations too   |
| intake-live         | Validate signup applications and protect stored answers             | Remove dependent signup operations too   |
| code-snippets       | Build highlighted code examples from source files                   | Remove dependent content features too    |
| check-harness       | Run checks and report their results                                 | Remove dependent deployment checks too   |
| deploy              | Plan, apply, check, and roll back deployments                       | Remove dependent deployment commands too |
| web                 | Browser UI built with Foldkit                                       | Optional                                 |
| cli                 | Command-line interface                                              | Optional                                 |
| mischief            | Hosted site, agent interfaces, and code sandbox                     | Optional                                 |
| infra               | Alchemy cloud resource declarations                                 | Keep resources your app uses             |

A [cartridge](https://ratstack.sh/lore/cartridges) is a package for one job, including its implementation and cloud resources. Add it with one package and one line that provides the implementation. Removing it must leave unrelated packages working.

The hosted agent interfaces run in Mischief; extracting them into a separate cartridge is still work to do.

[Hexagonal architecture](https://ratstack.sh/lore/hexagonal-architecture) keeps service interfaces in core and provider-specific implementations outside it. A [port](https://ratstack.sh/lore/hexagonal-architecture) names a job; an [adapter](https://ratstack.sh/lore/hexagonal-architecture) connects it to a provider. [HATEOAS](https://ratstack.sh/lore/hateoas) uses response links to guide the next action. [Analytics](https://ratstack.sh/systems/analytics) is a running capture system; [interest signup](https://ratstack.sh/systems/interest) shows consent and confirmation across a delivery adapter.

## One capability, several interfaces

A [contract](https://ratstack.sh/systems/capabilities) defines an operation's name, input, output, and errors. A [capability](https://ratstack.sh/systems/capabilities) binds that contract to a server-side handler. A [projection](https://ratstack.sh/lore/one-capability-every-surface) exposes the capability through one interface, such as the CLI (command-line interface) or HTTP.

XState models work as states and the transitions between them. Effect Schema defines and validates the data each operation accepts and returns.

[RPC (remote procedure call)](https://ratstack.sh/lore/one-capability-every-surface) lets the browser call server operations. [OpenAPI](https://ratstack.sh/lore/openapi-describes-the-refusal-too) describes the HTTP API in a format that software can read. Code mode is a way to run a program that calls tools inside a sandbox, a restricted execution environment.

Diagram: One capability projected to the command line, HTTP with OpenAPI, MCP tools, browser RPC, and sandbox code mode

```text
              ┌─────────────────────────────────────┐
              │  one contract + capability          │
              │  Effect Schema: input · output · err│
              │  one Effect handler                 │
              │  XState when the work has states    │
              └──────────────────┬──────────────────┘
                                 │  defineContract → implement
    ┌────────────┬────────────┬────────────┬────────────┬────────────┐
    ▼            ▼            ▼            ▼            ▼
  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐
  │ command  │ │   HTTP   │ │   MCP    │ │   RPC    │ │ sandbox  │
  │   line   │ │ + OpenAPI│ │  tools   │ │ browser  │ │ code mode│
  └──────────┘ └──────────┘ └──────────┘ └──────────┘ └──────────┘

  checked by  TypeScript 7 · Oxlint · Vitest · lefthook
  shipped by  pnpm · Turborepo · Alchemy → Cloudflare Worker
```

All five projections share one contract and handler. Agents use MCP, HTTP, or code mode.

## The pattern in code

This is the whole search capability. Every interface calls it.

```ts
export const search = implement(searchContract, ({ limit, query }) =>
  ContentStore.use((store) => store.search(query, limit)).pipe(
    Effect.map((matches) => ({ matches, total: matches.length }))
  )
);
```

The schemas define the input, output, and errors. Each projection calls the same handler through the same contract.

## Learn the stack

Install these short guides for your agent with the command above, or read them here.

### Start here

- [rat-stack-mode](/skills/rat-stack-mode)
- [use-a-prompt](/skills/use-a-prompt)

### See how the pieces fit

- [find-peers](/skills/find-peers)
- [learn](/skills/learn)
- [learn-alchemy](/skills/learn-alchemy)
- [learn-rat-stack](/skills/learn-rat-stack)

### Learn by building

- [add-a-capability](/skills/add-a-capability)
- [add-a-lifecycle-machine](/skills/add-a-lifecycle-machine)
- [add-a-store](/skills/add-a-store)
- [query-design](/skills/query-design)
- [write-a-wiki-page](/skills/write-a-wiki-page)

### Choose what you keep

- [keep-or-cut](/skills/keep-or-cut)
- [uncomplect](/skills/uncomplect)

### Keep the fence sharp

- [gardener](/skills/gardener)

### Ship with evidence

- [investigate-a-failure](/skills/investigate-a-failure)
- [profile-a-worker](/skills/profile-a-worker)
- [ship](/skills/ship)
- [test-behaviour](/skills/test-behaviour)

XState 6 and Alchemy are pre-release. APIs move; [pins.md](https://ratstack.sh/pins.md) lists the exact versions this repo builds against.

## Source files

- [Agent rules for rat-stack (AGENTS.md)](/AGENTS.md)
- [Purpose and boundaries of rat-stack (VISION.md)](/VISION.md)
- [Build and run rat-stack (README.md)](/README.md)
- [Dependency vendoring rules (vendor/README.md)](/vendor/README.md)
- [Exact workspace dependency pins (pins.md)](/pins.md)
- [Change log](/log)
- [Source change history (log.md)](/log.md)
- [Lint and type escape ledger (debt.md)](/debt.md)
- [Effect 4 study: September 2026](/resources/effect-4-reference-projects)
- [Schema projections: 2026-09-18 history](/resources/schema-projections-and-code-mode)
- [Learn rat-stack from your own agent](/learn)
- [Oxlint rule limits](/resources/lint-rule-limits)
- [Effect + Alchemy peers](/resources/peers)
- [Effect + Alchemy peer patterns](/resources/same-version-repos)

## Lore on this page

- [CAFE names the Cloudflare, Alchemy, Foldkit and Effect stack](/lore/cafe-stack)
- [Auth](/systems/auth)
- [Analytics](/systems/analytics)
- [Double opt-in](/lore/double-opt-in)
- [Hosted agent interfaces](/systems/agent-front-door)
- [Cartridges](/lore/cartridges)
- [One program can replace several tool calls](/lore/one-program-can-replace-several-tool-calls)
- [MCP is another surface](/lore/mcp-is-another-surface)
