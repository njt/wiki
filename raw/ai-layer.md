---
url: https://gitlab.com/kriscodes2-claude-plugins/ai-layer
date_fetched: 2026-10-11
---

# Bootstrap AI Layer

A Claude Code plugin for developers and platform teams working in Node and TypeScript: it gives a
session a DuckDB brain that remembers across restarts, a TypeScript language server that answers
"where is this symbol and who calls it" without reading whole files, and a lifecycle of skills for
planning, review, CI and release. **88 skills and 25 agents** across three packs, measured
2026-09-07 against this tree.

Free and MIT: the packs, both MCP servers, and a local single-machine brain. Not free: the Brain
Gateway and the rest of the Koinessa platform around it. The full boundary, skill by skill, is
[docs/OPEN-CORE.md](docs/OPEN-CORE.md).

## Install

```text
/plugin marketplace add https://gitlab.com/kriscodes2-claude-plugins/ai-layer.git
/plugin install ai-layer-core@kriscodes2-claude-plugins
/ai-layer:doctor --fix
```

`ai-layer-marketing@kriscodes2-claude-plugins` and `ai-layer-infra@kriscodes2-claude-plugins`
install the same way and are optional; core never requires either.

Restart Claude Code, then run `/ai-layer:init` in each project.

A git install ships no `node_modules`, because `@duckdb/node-api` is a native binding and
`typescript-language-server` is a spawned package -- neither can be bundled into a plugin. Until
`/ai-layer:doctor --fix` has run once, both MCP servers answer in setup mode and say so. That one
command is the whole install.

## What's in each pack

| Pack                   | Contents                        | What it is for                                                   |
| ---------------------- | ------------------------------- | ---------------------------------------------------------------- |
| `ai-layer-core`        | 65 skills, 22 agents, 2 servers | Always enable this one. Everything below depends on it.          |
| `ai-layer-marketing`   | 20 skills, 3 agents             | Optional. Lead-gen, content, social and analytics work.          |
| `ai-layer-infra`       | 3 skills, 0 agents              | Optional. A self-hosted k3s cluster and its GitLab runner pools. |
| _measured 2026-09-07._ |                                 |                                                                  |

**Core** carries the two MCP servers (`duckdb-brain` and `typescript-lsp`), the retrieval-first
SOP that routes symbol lookups through the index instead of grep, and the lifecycle:
`/ai-layer:init`, `/ai-layer:create-prd`, `/ai-layer:plan-review`, `/ai-layer:quality-gate`,
`/ai-layer:ci-local`, `/ai-layer:fix-ci`, `/ai-layer:ship`, `/ai-layer:security-review`, plus
session memory (`/ai-layer:recall`, `/ai-layer:remember`, `/ai-layer:resume-work`) and the symbol
index (`/ai-layer:doxygen-lookup`, `/ai-layer:sync-symbols`). Run `/ai-layer:help` for the full
list.

**Marketing** adds lead discovery, outreach drafting, content generation and publishing, and the
analytics pulls behind them (`/ai-layer-marketing:lead-discover`,
`/ai-layer-marketing:content-create`, `/ai-layer-marketing:social-publish`,
`/ai-layer-marketing:analytics-pull`). Twelve of its twenty skills need no brain at all.

**Infra** is for people who already run a self-hosted cluster: cluster health and guarded probe
pods (`/ai-layer-infra:k3s-access`) and GitLab runner pool provisioning on it
(`/ai-layer-infra:setup-gitlab-runner`, `/ai-layer-infra:setup-remote-gitlab-runner`). Every
endpoint comes from your own environment; nothing is baked in.

## The brain, in three modes

One resolver (`hooks/lib/brain-config.mjs`) decides the mode, and both MCP servers, both write
paths and `/ai-layer:doctor` all read the same answer.

| Mode      | When it is chosen                                                  | What works in it                                                                                                                                                                                                   |
| --------- | ------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `gateway` | a `BRAIN_GATEWAY_TOKEN` resolves                                   | The brain MCP forwards to the Brain Gateway you configure: shared multi-session brain, coordination plane, hybrid search and graph. The gateway is Koinessa's commercial control plane and is not in this release. |
| `local`   | no token resolves, and the brain server's dependencies are on disk | A single-machine DuckDB brain: `brain_query`, `brain_schema`, `brain_recall`, transcript ingest at commit, and symbol sync. No hybrid search, no graph, no coordination.                                           |
| `setup`   | neither of the above                                               | One status tool per server, naming `/ai-layer:doctor --fix`. Nothing is silently degraded.                                                                                                                         |

Precedence is `gateway` then `local`, and it is not a fallback: a configured gateway that is
unreachable stays in `gateway` mode and its writers fail open with one named line, rather than
quietly switching you to a different brain.

```text
  Claude Code session (any repo)
        |
        |  .mcp.json (${CLAUDE_PLUGIN_ROOT})
        v
  mcp/brain/launch.mjs ------ hooks/lib/brain-config.mjs ------ mcp/lsp/launch.mjs
    |                          mode = gateway | local | setup        |
    |-- gateway: stdio <-> HTTP proxy to $BRAIN_GATEWAY_URL/mcp      |-- deps present:
    |-- local:   mcp/duckdb/dist/index.js, opening and closing       |     mcp/lsp/dist/index.js
    |            $CLAUDE_BRAIN_DB per call                           |-- else:
    |-- setup:   mcp/lib/setup-server.mjs                            |     mcp/lib/setup-server.mjs
```

That picture and this README are checked against the resolver by `tests/docs/readme.test.mjs`: a
fourth mode fails the test rather than shipping a stale diagram.

## Requirements

| Tool         | Minimum         | Why                                                                                                                    |
| ------------ | --------------- | ---------------------------------------------------------------------------------------------------------------------- |
| Claude Code  | current         | The plugin runtime.                                                                                                    |
| Node.js      | 20 or newer     | Both MCP servers, the hooks, and the test runner.                                                                      |
| git          | any             | The install is a git clone; several hooks read git state.                                                              |
| bash         | any             | `init.sh`. On Windows, the one from Git for Windows.                                                                   |
| network      | once            | `/ai-layer:doctor --fix` runs `npm ci` in each MCP directory. Nothing after.                                           |
| `typescript` | in your project | `typescript-language-server` loads the TypeScript your project installs; without it the LSP tools report no workspace. |
| `duckdb` CLI | optional        | Nothing here requires it. `/ai-layer:doctor` reports it as information only.                                           |

The `duckdb` CLI is genuinely optional: `local` mode exists only when the DuckDB binding that
ships with the brain server resolves, and that binding is the only writer, so there is no CLI
write path to fall back to. Install the CLI if you want a second way to open the same file by
hand; skip it otherwise.

### Configuration

| Variable              | Default                                  | What it controls                        |
| --------------------- | ---------------------------------------- | --------------------------------------- |
| `CLAUDE_BRAIN_DB`     | `~/ClaudeBrain/brain.duckdb` on every OS | Where the local brain file lives.       |
| `BRAIN_GATEWAY_TOKEN` | unset                                    | Set it, and the mode becomes `gateway`. |
| `BRAIN_GATEWAY_URL`   | `http://127.0.0.1:8787`                  | Which gateway, when a token resolves.   |
| `CLAUDE_PROJECT_ID`   | derived from `package.json`              | The project partition key in the brain. |
| `LSP_PROJECT_ROOT`    | set per project by `/ai-layer:init`      | Which project the LSP indexes.          |
| `LSP_WARMUP_FILES`    | common entry points                      | Files the LSP opens on startup.         |
| `AI_LAYER_HINT`       | unset                                    | `off` suppresses the setup-mode hint.   |

### Gateway mode

If a `BRAIN_GATEWAY_TOKEN` resolves -- from the process environment, a `.env.local` beside your
project or above it, or `~/.claude` -- the brain MCP stops opening a local file and forwards to
the Brain Gateway at `BRAIN_GATEWAY_URL` instead. That gateway is yours to run and configure. With
no token set, nothing in this plugin opens a network connection to anything.

### Adapting the CI template

`/ai-layer:init` places a `.gitlab-ci.yml` only when the project's `origin` is a GitLab host and
the file is absent; on any other remote it prints one line saying it placed nothing, and points
here. The template is a plain `node:20` pipeline (install, lint, typecheck, test, build,
doxygen-check) with a commented example of including a CI/CD Catalog component -- copy it to
whatever your CI runs and keep the job names, since `/ai-layer:ci-local` and `/ai-layer:fix-ci`
read them. Scripts that run outside a plugin session resolve the pack through
`AI_LAYER_PLUGIN_ROOT`.

## Troubleshooting

Every diagnostic reads `<scope> FINDING:` or `<scope> FAILED:` and names the next command.

| What you see                                                                         | What it means                                                                                                                       |
| ------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------- |
| `ai-layer: brain and/or lsp are in setup mode -- run /ai-layer:doctor --fix (once).` | The install has not materialized its runtime dependencies yet. Run it, then restart Claude Code. `AI_LAYER_HINT=off` silences this. |
| `Brain MCP is in setup mode: runtime deps are not installed.`                        | The same state, seen from the MCP tool surface. Same fix.                                                                           |
| `doctor FINDING: local-writes unavailable (<reason>) -- fix: /ai-layer:doctor --fix` | Reads work, writes would not. The brain would stop remembering, so it is reported rather than skipped.                              |
| `doctor FINDING: gateway configured but unreachable (2s)`                            | A token resolved but the gateway did not answer. Mode stays `gateway` and writes fail open. Start it, or unset the token.           |
| `doctor FINDING: brain has 0 rows in chat_history -- new brain`                      | Expected on day one. It fills as you work; nothing is broken.                                                                       |
| `doctor FAILED: npm ci in <dir> exited <code>`                                       | Run `npm ci` in that directory by hand and read npm's error. `--fix` never falls back to `npm install`.                             |
| `lsp_find_symbol` returns nothing                                                    | tsserver has not opened the right files. Set `LSP_WARMUP_FILES` to paths covering your major subtrees.                              |

`/ai-layer:doctor` with no flag prints the same findings without changing anything, and `--json`
emits them as `{"ok","mode","checks","findings"}` for a script. Neither ever prints a token.

## Privacy

- **Local mode reaches nothing.** The brain is a DuckDB file on your disk. Both MCP servers are
  local Node processes speaking stdio to Claude Code.
- **Gateway mode talks to exactly one host: the gateway you configured.** It is not our server
  unless you point it at one. Nothing else changes.
- The pre-commit hook reads your own session transcripts from `~/.claude/projects/` and writes
  rows to your brain. The Stop hook writes a review file into your repository and a row to your
  brain.
- Skills that call GitLab or GitHub do so through your own authenticated `glab` / `gh`.
- No analytics, no telemetry, no account, no phone-home, in any mode. If that ever changes, it
  will be stated here and in `CHANGELOG.md` with a version bump.

## License

MIT. See [LICENSE](LICENSE).

## Contributing

Issues are welcome, and so are merge requests.

Releases here are snapshots of a private monorepo, not the development history: each version
arrives as one commit and a `v<semver>` tag, with `CHANGELOG.md` as the public record. A merge
request is read, reviewed and re-applied upstream rather than merged into this tree, so your
change lands in the next release with attribution rather than being lost at the next snapshot.
There is no Claude Code marketplace submission; this git URL is the distribution.

To run the checks locally:

```bash
npm test           # pack manifests agree, and the public surface carries no private path
npm run test:full  # every node:test suite in the tree
```

`test:full` goes through `scripts/run-tests.mjs`, which discovers the suite files itself and hands
`node --test` an explicit list. That indirection is deliberate: Node 20 treats a directory argument
as a directory to walk, and Node 21 treats the same string as a glob that matches the directory
itself, so no single argument is correct on both. Node 20 remains the floor.

Built on [Claude Code](https://claude.ai/code), [DuckDB](https://duckdb.org/),
[typescript-language-server](https://github.com/typescript-language-server/typescript-language-server),
and the [Model Context Protocol](https://modelcontextprotocol.io/). Maintained by Kris Codes LLC.
