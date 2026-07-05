---
url: https://github.com/waldekmastykarz/atifact
date_fetched: 2026-07-05
backfilled: true
---

Convert agent logs to ATIF trajectories. One command. Zero dependencies.

Turn HAR files, Claude Code logs, Copilot CLI logs, and Codex CLI logs into standardized ATIF v1.7 trajectory JSON — ready for debugging, visualization, fine-tuning, and RL pipelines.

Give your AI coding agent the atifact skill so it can extract trajectories on your behalf:

`npx skills add waldekmastykarz/atifact`Once installed, ask your agent to *"extract the trajectory from this HAR file"*, *"convert Claude Code logs to ATIF"*, *"convert Copilot CLI logs"*, or *"convert Codex CLI logs"* and it will handle the rest.

`npm install -g atifact````
# Convert a HAR file (auto-detected)
atifact session.har
# Convert Claude Code logs
atifact claude-log.jsonl
# Convert Copilot CLI logs
atifact copilot-session.jsonl
# Convert Codex CLI logs
atifact codex-session.jsonl
# Pipe to stdout (returns JSON array of trajectories)
atifact session.har --json | jq '.steps | length'
```
Output: `<input>.trajectory.json` in ATIF v1.7 format. Copilot CLI and Codex CLI logs with subagents produce additional `<input>.trajectory.<name>.json` files.

`--json` mode outputs a single trajectory with subagents embedded in the `subagent_trajectories` array to stdout with no files written.

| Format | Source | Flag | 
|---|---|---|
| HAR | OpenAI Chat Completions API, OpenAI Responses API, Anthropic Messages API | `har` | 
| JSONL | Claude Code CLI session logs | `claude-code-jsonl` | 
| JSONL | Copilot CLI session logs | `copilot-cli-jsonl` | 
| JSONL | Codex CLI `exec --json`logs | `codex-cli-jsonl` | 

Format is auto-detected from file contents (not extension). Force it with `-f`:

```
atifact myfile.log -f claude-code-jsonl
atifact myfile.log -f copilot-cli-jsonl
atifact myfile.log -f codex-cli-jsonl
```
```
atifact <input-file> [options]
```
| Option | Description | 
|---|---|
| `-o, --output <prefix>` | Output path prefix (default: input file path). Main: `<prefix>.trajectory.json`, subagents:`<prefix>.trajectory.<name>.json` | 
| `-f, --format <fmt>` | Force input format: `har`,`claude-code-jsonl`,`copilot-cli-jsonl`,`codex-cli-jsonl` | 
| `--json` | Write trajectory to stdout with subagents embedded (no files written) | 
| `-q, --quiet` | Suppress progress messages | 
| `-h, --help` | Show help | 
| `--version` | Print version | 

| Code | Meaning | 
|---|---|
| `0` | Success | 
| `1` | Runtime error (parse failure, I/O error) | 
| `2` | Invalid usage (bad arguments, missing file) | 

atifact produces ATIF v1.7 JSON with:

- **Steps**— user messages, agent responses, tool calls, and observations
- **Metrics**— token counts, costs, cached tokens per step
- **Tool calls**— structured function name + arguments with observation results
- **Subagent trajectories**— Copilot CLI and Codex CLI subagents produce separate trajectory files linked via- `subagent_trajectory_ref`with- `trajectory_id`resolution;- `--json`mode embeds them in- `subagent_trajectories`
- **Final metrics**— aggregated totals across the trajectory
- All timestamps preserved as ISO 8601 from source data
- Null/undefined fields excluded for compact output

```
atifact recording.har -o my-trajectory
# Writes: my-trajectory.trajectory.json
```
`atifact ~/.claude/projects/*/sessions/*.jsonl --json --quiet > trajectory.json````
atifact copilot-session.jsonl
# Writes: copilot-session.jsonl.trajectory.json
# Writes: copilot-session.jsonl.trajectory.<subagent-name>.json (per subagent)
```
```
atifact codex-session.jsonl
# Writes: codex-session.jsonl.trajectory.json
```
`atifact session.har --json | jq '[.steps[] | select(.source == "agent")] | length'`- Node.js 22+
