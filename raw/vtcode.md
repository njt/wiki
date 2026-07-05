---
url: https://github.com/vinhnx/VTCode
date_fetched: 2026-07-05
backfilled: true
---

VT Code is a Rust coding agent built for long-running autonomous workflows, with OS-native sandboxing, multi-provider LLM support, open protocols, and extensible Skills.

- **Agent runtime**- Interactive TUI, slash commands, streaming,- `ask`/- `exec`CLI, session resume
- **Coding tools**- Safe file ops, ripgrep search + ast-grep outline symbol maps, fuzzy discovery, code intelligence, project indexing, terminal execution
- **Extensibility**- Agent Skills, MCP client/server, lifecycle hooks, subagents, custom providers, Zed ACP, VS Code, Claude Code
- **Model providers**- 21+ LLM providers: Anthropic, OpenAI, Gemini, OpenRouter, Ollama, LM Studio, and more
- **Safety**- Restricted shell sandbox, tool guardrails, subprocess isolation, full audit logging
- **Protocols**- Open Responses, Agent2Agent (A2A), ATIF, Anthropic Messages API
- **Loop engineering**- Worktree isolation for parallel agents, propose/verify sub-agent separation, durable loop state, cost guardrails

```
curl -fsSL https://raw.githubusercontent.com/vinhnx/vtcode/main/scripts/install.sh | bash
vtcode init
vtcode
```
- **Install**- one-liner for macOS/Linux (also installs ripgrep + ast-grep)
- **Init**- scaffolds- `vtcode.toml`,- `.vtcode/`, and- `AGENTS.md`in your project
- **Launch**- interactive TUI with streaming, tools, and slash commands

```
vtcode                         # interactive TUI
vtcode init                    # scaffold project config + AGENTS.md
vtcode ask "explain Rc vs Arc" # one-shot question
vtcode exec "refactor main.rs" # headless task with full tool access
vtcode review                  # review uncommitted changes
vtcode update                  # self-update
```
- **Interactive TUI**- Primary agents, slash commands (- `/model`,- `/review`,- `/mcp`,- `/skills`,- `/theme`,- `/compact`)
- **Full automation**-- `--full-auto`CLI, plan-build-evaluate harness, subagents, scheduled tasks
- **Providers**- Setup guides for all 21 providers
- **Configuration**-- `vtcode.toml`, tool config, lifecycle hooks
- **Agent Skills**- Creating, loading, and sharing skills
- **MCP Integration**- Client and server modes
- **Editor guides**- Zed ACP, VS Code, Claude Code
- **Safety**- Shell sandbox, security hardening, threat model
- **Protocols**- Open Responses, ATIF, A2A, Anthropic Messages API
- **Loop engineering**- Worktree isolation, propose/verify, loop state, cost guardrails

VT Code supports 21+ LLM providers out of the box, plus any OpenAI-compatible API via `[[custom_providers]]`.

| Category | Providers | 
|---|---|
| Cloud LLMs | Anthropic · OpenAI · Gemini · Z.AI · Moonshot (Kimi) · StepFun · MiniMax · Mistral · Qwen | 
| Gateways | OpenRouter · Evolink · HuggingFace · Atlas Cloud | 
| Local inference | Ollama · LM Studio · llama.cpp | 
| Other | GitHub Copilot · Anthropic API Compat · Poolside | 

Read: Provider Guides.

```
git clone https://github.com/vinhnx/vtcode.git
cd vtcode
./scripts/run-debug.sh
```
Rust stable, edition 2024, MSRV 1.88. Workspace of ~30 crates:

| Layer | Crates | 
|---|---|
| Binary | `vtcode` | 
| Core | `vtcode-core`,`vtcode-ui`,`vtcode-config` | 
| Extracted | `vtcode-llm`,`vtcode-skills`,`vtcode-safety`,`vtcode-a2a`,`vtcode-mcp` | 
| Infrastructure | `vtcode-commons`,`vtcode-auth`,`vtcode-bash-runner`,`vtcode-macros` | 
| Tooling | `vtcode-indexer`,`vtcode-exec-events` | 
| Protocol | `vtcode-acp`,`vtcode-utility-tool-specs` | 

```
./scripts/check-dev.sh  # fast quality gate (clippy, fmt, check)
cargo nextest run        # parallel test runner
```
I'd love to have you, bug reports, docs, features, ideas, all welcome. Start with issues or good first issues. AI agents see AGENTS.md. Humans see CONTRIBUTING.md.

VT Code is a labor of love built in my spare time. If it's helped you ship something or learn something, a sponsorship would mean the world.
