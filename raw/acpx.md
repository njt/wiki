---
url: https://github.com/openclaw/acpx
date_fetched: 2026-07-05
backfilled: true
---

⚠️ `acpx`is in alpha and the CLI/runtime interfaces are likely to change. Anything you build downstream of this might break until it stabilizes.

ACP coverage status: see ACP Spec Coverage Roadmap.


Your agents love acpx! 🤖❤️ They hate having to scrape characters from a PTY session 😤

`acpx` is a headless CLI client for the Agent Client Protocol (ACP), so AI agents and orchestrators can talk to coding agents over a structured protocol instead of PTY scraping.

One command surface for Pi, OpenClaw ACP, Codex, Claude, and other ACP-compatible agents. Built for agent-to-agent communication over the command line.

- **Persistent sessions**: multi-turn conversations that survive across invocations, scoped per repo
- **Named sessions**: run parallel workstreams in the same repo (- `-s backend`,- `-s frontend`)
- **Prompt queueing**: submit prompts while one is already running, they execute in order
- **Cooperative cancel command**:- `cancel`sends ACP- `session/cancel`via queue IPC without tearing down session state
- **Soft-close lifecycle**: close sessions without deleting history from disk
- **Queue owner TTL**: keep queue owners alive briefly for follow-up prompts (- `--ttl`)
- **Fire-and-forget**:- `--no-wait`queues a prompt and returns immediately
- **Graceful cancel**:- `Ctrl+C`sends ACP- `session/cancel`before force-kill fallback
- **Session controls**:- `set-mode`and- `set <key> <value>`for- `session/set_mode`and- `session/set_config_option`
- **Crash reconnect**: dead agent processes are detected and sessions are resumed or reloaded automatically
- **Prompt from file/stdin**:- `--file <path>`or piped stdin for prompt content
- **Config files**: global + project JSON config with- `acpx config show|init`
- **Session inspect/history**:- `sessions show`and- `sessions history --limit <n>`
- **Session export/import**: move portable session archives between machines
- **Local status checks**:- `status`reports running/idle/dead/no-session, pid, uptime, last prompt
- **Client methods**: stable- `fs/*`and- `terminal/*`handlers with permission controls and cwd sandboxing
- **Auth handshake**: stable- `authenticate`support via env/config credentials
- **Structured output**: typed ACP messages (thinking, tool calls, diffs) instead of ANSI scraping
- **Any ACP agent**: built-in registry +- `--agent`escape hatch for custom servers
- **One-shot mode**:- `exec`for stateless fire-and-forget tasks
- **Compare across agents**:- `acpx compare pi openclaw codex 'fix the bug'`runs the same one-shot prompt against multiple ACP-compatible agents and summarizes timing, token usage, permissions, and final output side by side
- **Experimental flows**:- `flow run <file>`for TypeScript workflow modules over multiple prompts
- **Runtime-owned flow actions**: shell-backed action steps can prepare workspaces and other deterministic mechanics outside the agent turn
- **Flow workspace isolation**:- `acp`nodes can target an explicit per-step cwd, so flows can keep agent work inside disposable worktrees

```
$ acpx codex sessions new
$ acpx codex "find the flaky test and fix it"
[thinking] Investigating test suite for flaky failures
[tool] Run npm test -- --reporter=verbose (running)
[tool] Run npm test -- --reporter=verbose (completed)
  output:
    ✓ auth.login (0.8s)
    ✗ checkout.submit (timed out after 5000ms)
    ✓ cart.add (0.3s)
[thinking] Found it — checkout.submit has a race condition in the async setup
[tool] Edit src/checkout.test.ts (completed)
  output:
    Success. Updated 1 file.
[tool] Run npm test -- checkout.submit (completed)
  output:
    ✓ checkout.submit (0.4s)
Fixed: added `await` to the setup hook in checkout.submit. The test was
reading stale state from the previous run.
[done] end_turn
```
Copy the block below and paste it into your Pi, OpenClaw, Claude Code, or similar agent harness. It will install acpx, read the skill reference, and know how to use ACP for all future coding agent tasks.

```
I want you to use acpx to run coding agents over the Agent Client Protocol
(ACP) instead of scraping PTY sessions. Please do the following:
1. Install acpx globally (recommended) or use npx:
   npm install -g acpx@latest
   Or without installing:
   npx acpx@latest
2. For Pi or OpenClaw, use the reference URL below. For Codex-style skill
   installation, install the acpx skill so you have the full reference
   available:
   npx acpx@latest --skill install acpx --agent codex --scope user
   Use --agent claude for Claude Code. For another harness not listed by
   --skill install --help, use the reference URL below instead.
3. Read the acpx skill reference so you know every command, flag, and
   workflow pattern:
   https://raw.githubusercontent.com/openclaw/acpx/main/skills/acpx/SKILL.md
4. If you need the full CLI reference with all options and examples:
   https://raw.githubusercontent.com/openclaw/acpx/main/docs/CLI.md
From now on, when you need to delegate work to a coding agent, use acpx
instead of spawning raw terminal sessions. For example:
  npx acpx@latest codex "fix the failing tests"
  npx acpx@latest claude "refactor the auth module"
  npx acpx@latest codex exec "one-shot: summarize this repo"
```
`npm install -g acpx@latest`Or run without installing:

`npx acpx@latest codex "fix the tests"`Session state lives in `~/.acpx/` either way. Global install is a little faster, but `npx acpx@latest` works fine.

`acpx` auto-downloads ACP adapters with `npx` on first use. You do not need to install adapter packages manually.

The only prerequisite is the underlying coding agent you want to use:

- `acpx pi`-> Pi Coding Agent: https://github.com/mariozechner/pi
- `acpx openclaw`-> OpenClaw ACP bridge: https://github.com/openclaw/openclaw
- `acpx codex`-> Codex CLI: https://codex.openai.com
- `acpx claude`-> Claude Code: https://claude.ai/code

Additional built-in agent docs live in agents/README.md.

```
acpx codex sessions new                        # create a session (explicit) for this project dir
acpx codex 'fix the tests'                     # implicit prompt (routes via directory-walk)
acpx codex prompt 'fix the tests'              # explicit prompt subcommand
echo 'fix flaky tests' | acpx codex            # prompt from stdin
acpx codex --file prompt.md                    # prompt from file
acpx codex --file - "extra context"            # explicit stdin + appended args
acpx codex --no-wait 'draft test migration plan' # enqueue without waiting if session is busy
acpx codex cancel                               # cooperative cancel of in-flight prompt
acpx codex set-mode auto                        # session/set_mode (adapter-defined mode id)
acpx codex set model 'gpt-5.2[high]'            # adapter-advertised model control
acpx exec 'summarize this repo'                # default agent shortcut (codex)
acpx codex exec 'what does this repo do?'      # one-shot, no saved session
acpx codex sessions new --name api              # create named session
acpx codex -s api 'implement token pagination'  # prompt in named session
acpx codex sessions new --name docs             # create another named session
acpx codex -s docs 'rewrite API docs'           # parallel work in another named session
acpx codex sessions              # list sessions for codex command
acpx codex sessions list         # explicit list
acpx codex sessions show         # inspect cwd session metadata
acpx codex sessions history      # show recent turn history
acpx codex sessions new          # create fresh cwd-scoped default session
acpx codex sessions new --name api # create fresh named session
acpx codex sessions ensure       # return existing scoped session or create one
acpx codex sessions ensure --name api # ensure named scoped session
acpx codex sessions close        # close cwd-scoped default session
acpx codex sessions close api    # close cwd-scoped named session
acpx codex status                # local process status for current session
acpx config show                 # show resolved config (global + project)
acpx config init                 # create ~/.acpx/config.json template
```
Main landing harness examples:

```
acpx pi 'review recent changes'
acpx openclaw exec 'summarize active session state' # built-in OpenClaw ACP bridge
acpx codex 'fix the failing typecheck'
acpx claude 'refactor auth middleware' # built-in claude agent
```
Additional supported harnesses and their specific notes are documented in agents/README.md.

```
acpx my-agent 'review this patch'                      # unknown name -> raw command
acpx --agent './bin/dev-acp --profile ci' 'run checks' # --agent escape hatch
```
```
# Review a PR in a dedicated session and auto-approve permissions
acpx --cwd ~/repos/shop --approve-all codex -s pr-842 \
  'Review PR #842 for regressions and propose a minimal fix'
# Keep parallel streams for the same repo
acpx codex -s bugfix 'isolate flaky checkout test'
acpx codex -s release 'draft release notes from recent commits'
```
```
acpx --approve-all codex 'apply the patch and run tests'
acpx --approve-reads codex 'inspect repo structure and suggest plan' # default mode
acpx --deny-all codex 'explain what you can do without tool access'
acpx --non-interactive-permissions fail codex 'fail instead of deny in non-TTY'
acpx --policy '{"escalate":["execute"],"defaultAction":"deny"}' --format json codex exec 'ask before shell'
acpx --cwd ~/repos/backend codex 'review recent auth changes'
acpx --format text codex 'summarize your findings'
acpx --format json codex exec 'review changed files'
acpx --format json --json-strict codex exec 'machine-safe JSON only'
acpx flow run ./my-flow.ts --input-file ./flow-input.json
acpx --timeout 1800 flow run ./my-flow.ts
acpx --format quiet codex 'final recommendation only'
acpx --suppress-reads codex exec 'show tool activity without dumping file bodies'
acpx --timeout 90 codex 'investigate intermittent test timeout'
acpx --ttl 30 codex 'keep queue owner alive for quick follow-ups'
acpx --verbose codex 'debug why adapter startup is failing'
```
`acpx flow run <file>` executes a TypeScript flow module through the `acpx/flows`
runtime and persists run state under `~/.acpx/flows/runs/`.

Flows are for multi-step ACP work where one prompt is not enough:

- `acp`steps keep model-shaped work in ACP
- `decision()`and- `decisionEdge()`wrap constrained-choice ACP branching without adding a new node type
- `action`steps handle deterministic mechanics like shell commands or GitHub calls
- `compute`steps do local routing or shaping
- `checkpoint`steps pause for something outside the runtime

The source tree includes flow examples under examples/flows/README.md:

- small examples such as `echo`,`branch`,`shell`,`workdir`, and`two-turn`
- a larger PR-triage example under examples/flows/pr-triage/README.md
- a replay viewer under examples/flows/replay-viewer/README.md for inspecting saved run bundles in the browser

Example runs:

```
acpx flow run ./my-flow.ts --input-file ./flow-input.json
acpx flow run examples/flows/branch.flow.ts \
  --input-json '{"task":"FIX: add a regression test for the reconnect bug"}'
acpx flow run examples/flows/pr-triage/pr-triage.flow.ts \
  --input-json '{"repo":"openclaw/acpx","prNumber":150}'
```
The PR-triage example is only an example workflow. It can comment on or close real GitHub PRs if you run it against a live repository.

`acpx` reads config in this order (later wins):

- global: `~/.acpx/config.json`
- project: `<cwd>/.acpxrc.json`

CLI flags always win over config values.

Use `--mcp-config <path>` to load only the `mcpServers` array from an external JSON file without
writing a project config file. Relative paths resolve from `--cwd`.

Supported keys:

```
{
  "defaultAgent": "codex",
  "defaultPermissions": "approve-all",
  "nonInteractivePermissions": "deny",
  "authPolicy": "skip",
  "ttl": 300,
  "timeout": null,
  "format": "text",
  "agents": {
    "my-custom": { "command": "./bin/my-acp-server", "args": ["acp"] }
  },
  "auth": {
    "my_auth_method_id": "credential-value"
  }
}
```
Use `acpx config show` to inspect the resolved result and `acpx config init` to create the global template.

For ACP `authenticate` handshakes, use either config `auth` entries or explicit
`ACPX_AUTH_<METHOD_ID>` environment variables such as `ACPX_AUTH_OPENAI_API_KEY`.
Ambient provider env vars such as `OPENAI_API_KEY` are still passed through to
child agents, but they do not trigger ACP auth-method selection on their own.

```
# text (default): human-readable stream with tool updates
acpx codex 'review this PR'
# json: NDJSON events, useful for automation
acpx --format json codex exec 'review this PR' \
  | jq -r 'select(.type=="tool_call") | [.status, .title] | @tsv'
# json-strict: suppresses non-JSON stderr output (requires --format json)
acpx --format json --json-strict codex exec 'review this PR'
# quiet: final assistant text on stdout; one structured stderr line on failure
acpx --format quiet codex 'give me a 3-line summary'
# suppress read payloads while keeping the selected output format
acpx --suppress-reads codex exec 'inspect the repo and report tool usage'
```
- `text`: human-readable stream with assistant text and tool updates
- `json`: raw ACP NDJSON stream for automation
- `quiet`: final assistant text on stdout; failed prompts emit one structured- `[acpx] error:`line on stderr
- `--suppress-reads`: replace raw read-file contents with- `[read output suppressed]`in- `text`and- `json`output

JSON events include a stable envelope for correlation:

```
{
  "eventVersion": 1,
  "sessionId": "abc123",
  "requestId": "req-42",
  "seq": 7,
  "stream": "prompt",
  "type": "tool_call"
}
```
Session-control JSON payloads (`sessions new|ensure`, `status`) always include
`acpxRecordId` and `acpxSessionId`. They include `agentSessionId` only when the
adapter exposes a provider-native session ID. The text/quiet session id is the
local acpx record id; do not assume it can be passed to the native provider CLI
unless `agentSessionId` is present.

Built-ins:

| Agent | Adapter | Wraps | 
|---|---|---|
| `pi` | pi-acp | Pi Coding Agent | 
| `openclaw` | native ( `openclaw acp`) | OpenClaw ACP bridge | 
| `codex` | codex-acp | Codex CLI | 
| `claude` | claude-agent-acp | Claude Code | 
| `gemini` | native ( `gemini --acp`) | Gemini CLI | 
| `cursor` | native ( `cursor-agent acp`) | Cursor CLI | 
| `copilot` | native ( `copilot --acp --stdio`) | GitHub Copilot CLI | 
| `droid` | native ( `droid exec --output-format acp`) | Factory Droid | 
| `fast-agent` | `uvx fast-agent-mcp acp` | fast-agent | 
| `grok-build` | native ( `grok agent stdio`) | Grok Build | 
| `iflow` | native ( `iflow --experimental-acp`) | iFlow CLI | 
| `kilocode` | `npx -y @kilocode/cli acp` | Kilocode | 
| `kimi` | native ( `kimi acp`) | Kimi CLI | 
| `kiro` | native ( `kiro-cli-chat acp`) | Kiro CLI | 
| `mux` | `npx -y mux@^0.27.0 acp` | Mux | 
| `opencode` | `npx -y opencode-ai acp` | OpenCode | 
| `qoder` | native ( `qodercli --acp`) | Qoder CLI | 
| `qwen` | native ( `qwen --acp`) | Qwen Code | 
| `trae` | native ( `traecli acp serve`) | Trae CLI | 

`factory-droid` and `factorydroid` also resolve to the built-in `droid` adapter.

Additional built-in agent docs live in agents/README.md.

Use `--agent` as an escape hatch for custom ACP servers:

`acpx --agent ./my-custom-acp-server 'do something'`For repo-local OpenClaw checkouts, override the built-in command in config so `acpx openclaw ...`
spawns the ACP bridge directly without `pnpm` wrapper noise:

```
{
  "agents": {
    "openclaw": {
      "command": "env OPENCLAW_HIDE_BANNER=1 OPENCLAW_SUPPRESS_NOTES=1 node scripts/run-node.mjs acp --url ws://127.0.0.1:18789 --token-file ~/.openclaw/gateway.token --session agent:main:main"
    }
  }
}
```
- Prompt commands require an existing saved session record (created via `sessions new`or`sessions ensure`).
- Prompts route by walking up from `cwd`(or`--cwd`) to the nearest git root (inclusive) and selecting the nearest active session matching`(agent command, dir, optional name)`.
- If no git root is found, prompts only match an exact `cwd`session (no parent-directory walk).
- `-s <name>`selects a parallel named session during that directory walk.
- `sessions new [--name <name>]`creates a fresh session for that scope and soft-closes the prior one.
- `sessions ensure [--name <name>]`is idempotent: it returns an existing scoped session or creates one when missing.
- `sessions close [name]`soft-closes the session: queue owner/processes are terminated, record is kept with- `closed: true`.
- `sessions list`uses agent-side ACP- `session/list`when available; use- `--cursor`,- `--filter-cwd`, or- `--local`for pagination, cwd filtering, or saved-record inspection.
- Auto-resume for cwd scope skips sessions marked closed.
- Prompt submissions are queue-aware per session. If a prompt is already running, new prompts are queued and drained by the running `acpx`process.
- Queue owners use an idle TTL (default 300s). `--ttl <seconds>`overrides it;`--ttl 0`keeps owners alive indefinitely.
- `--no-wait`submits to that queue and returns immediately.
- `cancel`sends cooperative- `session/cancel`to the running queue owner process and returns success when no prompt is running (- `nothing to cancel`).
- `set-mode`and- `set`route through queue-owner IPC when active, otherwise they reconnect directly to apply- `session/set_mode`and- `session/set_config_option`.
- `<mode>`values for- `set-mode`are adapter-defined; unsupported values are rejected by the adapter (commonly- `Invalid params`).
- `exec`is always one-shot and does not reuse saved sessions.
- Session metadata is stored under `~/.acpx/sessions/`.
- Each successful prompt appends lightweight turn history previews (`role`,`timestamp`,`textPreview`) to session metadata.
- `Ctrl+C`during a running turn sends ACP- `session/cancel`and waits briefly for- `stopReason=cancelled`before force-killing if needed.
- If a saved session pid is dead on the next prompt, `acpx`respawns the agent, attempts`session/resume`when advertised or`session/load`otherwise, and transparently falls back to`session/new`if reconnecting fails.

See docs/CLI.md.

MIT
