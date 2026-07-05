---
url: https://github.com/thedotmack/claude-mem
date_fetched: 2026-07-05
backfilled: true
---

🇨🇳 中文 • 🇹🇼 繁體中文 • 🇯🇵 日本語 • 🇵🇹 Português • 🇧🇷 Português • 🇰🇷 한국어 • 🇪🇸 Español • 🇩🇪 Deutsch • 🇫🇷 Français • 🇮🇱 עברית • 🇸🇦 العربية • 🇷🇺 Русский • 🇵🇱 Polski • 🇨🇿 Čeština • 🇳🇱 Nederlands • 🇹🇷 Türkçe • 🇺🇦 Українська • 🇻🇳 Tiếng Việt • 🇵🇭 Tagalog • 🇮🇩 Indonesia • 🇹🇭 ไทย • 🇮🇳 हिन्दी • 🇧🇩 বাংলা • 🇵🇰 اردو • 🇷🇴 Română • 🇸🇪 Svenska • 🇮🇹 Italiano • 🇬🇷 Ελληνικά • 🇭🇺 Magyar • 🇫🇮 Suomi • 🇩🇰 Dansk • 🇳🇴 Norsk

#### Persistent memory compression system built for Claude Code.

|  |  | 

Quick Start • How It Works • Search Tools • Documentation • Configuration • Troubleshooting • License

Claude-Mem seamlessly preserves context across sessions by automatically capturing tool usage observations, generating semantic summaries, and making them available to future sessions. This enables Claude to maintain continuity of knowledge about projects even after sessions end or reconnect.

Install with a single command:

`npx claude-mem install`Or install for OpenCode:

`npx claude-mem install --ide opencode`Or install for Antigravity CLI (setup guide):

`npx claude-mem install --ide antigravity`Or install from the plugin marketplace inside Claude Code:

```
/plugin marketplace add thedotmack/claude-mem
/plugin install claude-mem
```
Restart Claude Code. Context from previous sessions will automatically appear in new sessions.


Note:Claude-Mem is also published on npm, but`npm install -g claude-mem`installs theSDK/library only— it does not register the plugin hooks or set up the worker service. Always install via`npx claude-mem install`or the`/plugin`commands above.

Install claude-mem as a persistent memory plugin on OpenClaw gateways with a single command:

`curl -fsSL https://install.cmem.ai/openclaw.sh | bash`The installer handles dependencies, plugin setup, AI provider configuration, worker startup, and optional real-time observation feeds to Telegram, Discord, Slack, and more. See the OpenClaw Integration Guide for details.

**Key Features:**

- 🧠 **Persistent Memory**- Context survives across sessions
- 📊 **Progressive Disclosure**- Layered memory retrieval with token cost visibility
- 🔍 **Skill-Based Search**- Query your project history with mem-search skill
- 🖥️ **Web Viewer UI**- Real-time memory stream at http://localhost:37777
- 💻 **Claude Desktop Skill**- Search memory from Claude Desktop conversations
- 🔒 **Privacy Control**- Use`<private>`tags to exclude sensitive content from storage
- ⚙️ **Context Configuration**- Fine-grained control over what context gets injected
- 🤖 **Automatic Operation**- No manual intervention required
- 🔗 **Citations**- Reference past observations with IDs (access via http://localhost:37777/api/observation/{id} or view all in the web viewer at http://localhost:37777)
- 🧪 **Beta Channel**- Try experimental features like Endless Mode via version switching

📚 **View Full Documentation** - Browse on official website

- **Installation Guide**- Quick start & advanced installation
- **Usage Guide**- How Claude-Mem works automatically
- **Search Tools**- Query your project history with natural language
- **Beta Features**- Try experimental features like Endless Mode

- **Context Engineering**- AI agent context optimization principles
- **Progressive Disclosure**- Philosophy behind Claude-Mem's context priming strategy

- **Overview**- System components & data flow
- **Architecture Evolution**- The journey from v3 to v5
- **Hooks Architecture**- How Claude-Mem uses lifecycle hooks
- **Hooks Reference**- 7 hook scripts explained
- **Worker Service**- HTTP API & Bun management
- **Database**- SQLite schema & FTS5 search
- **Search Architecture**- Hybrid search with Chroma vector database

- **Configuration**- Environment variables & settings
- **Development**- Building, testing, contributing
- **Troubleshooting**- Common issues & solutions

**Core Components:**

- **5 Lifecycle Hooks**- SessionStart, UserPromptSubmit, PostToolUse, Stop, SessionEnd (6 hook scripts)
- **Smart Install**- Cached dependency checker (pre-hook script, not a lifecycle hook)
- **Worker Service**- HTTP API on port 37777 with web viewer UI and 10 search endpoints, managed by Bun
- **SQLite Database**- Stores sessions, observations, summaries
- **mem-search Skill**- Natural language queries with progressive disclosure
- **Chroma Vector Database**- Hybrid semantic + keyword search for intelligent context retrieval

See Architecture Overview for details.

Claude-Mem provides intelligent memory search through **4 MCP tools** following a token-efficient **3-layer workflow pattern**:

**The 3-Layer Workflow:**

- `search`
- `timeline`
- `get_observations`

**How It Works:**

- Claude uses MCP tools to search your memory
- Start with `search`to get an index of results
- Use `timeline`to see what was happening around specific observations
- Use `get_observations`to fetch full details for relevant IDs
- **~10x token savings**by filtering before fetching details

**Available MCP Tools:**

- `search`
- `timeline`
- `get_observations`

**Example Usage:**

```
// Step 1: Search for index
search(query="authentication bug", type="bugfix", limit=10)
// Step 2: Review index, identify relevant IDs (e.g., #123, #456)
// Step 3: Fetch full details
get_observations(ids=[123, 456])
```
See Search Tools Guide for detailed examples.

Claude-Mem offers a **beta channel** with experimental features like **Endless Mode** (biomimetic memory architecture for extended sessions). Switch between stable and beta versions from the web viewer UI at http://localhost:37777 → Settings.

See **Beta Features Documentation** for details on Endless Mode and how to try it.

- **Node.js**: 20.0.0 or higher
- **Claude Code**: Latest version with plugin support
- **Bun**: JavaScript runtime and process manager (auto-installed if missing)
- **uv**: Python package manager for vector search (auto-installed if missing)
- **SQLite 3**: For persistent storage (bundled)

If you see an error like:

`npm : The term 'npm' is not recognized as the name of a cmdlet`Make sure Node.js and npm are installed and added to your PATH. Download the latest Node.js installer from https://nodejs.org and restart your terminal after installation.

Settings are managed in `~/.claude-mem/settings.json` (auto-created with defaults on first run). Configure AI model, worker port, data directory, log level, and context injection settings.

See the **Configuration Guide** for all available settings and examples.

Claude-Mem supports multiple workflow modes and languages via the `CLAUDE_MEM_MODE` setting.

This option controls both:

- The workflow behavior (e.g. code, chill, investigation)
- The language used in generated observations

Edit your settings file at `~/.claude-mem/settings.json`:

```
{
  "CLAUDE_MEM_MODE": "code--zh"
}
```
Modes are defined in `plugin/modes/`. To see all available modes locally:

`ls ~/.claude/plugins/marketplaces/thedotmack/plugin/modes/`| Mode | Description | 
|---|---|
| `code` | Default English mode | 
| `code--zh` | Simplified Chinese mode | 
| `code--ja` | Japanese mode | 

Language-specific modes follow the pattern `code--[lang]` where `[lang]` is the ISO 639-1 language code (e.g., `zh` for Chinese, `ja` for Japanese, `es` for Spanish).

Note:

`code--zh`(Simplified Chinese) is already built-in — no additional installation or plugin update is required.

See the **Development Guide** for build instructions, testing, and contribution workflow.

If experiencing issues, describe the problem to Claude and the troubleshoot skill will automatically diagnose and provide fixes.

See the **Troubleshooting Guide** for common issues and solutions.

Create comprehensive bug reports with the automated generator:

```
cd ~/.claude/plugins/marketplaces/thedotmack
npm run bug-report
```
Contributions are welcome! Please:

- Fork the repository
- Create a feature branch
- Make your changes with tests
- Update documentation
- Submit a Pull Request

See Development Guide for contribution workflow.

Claude-Mem is licensed under the Apache License 2.0.

We chose Apache-2.0 because durable agentic memory should be easy to embed in developer tools, local agents, MCP servers, enterprise systems, robotics stacks, and production agent harnesses.

See the LICENSE file for full details. See docs/license.md and docs/ip-boundary.md for licensing scope and the open/commercial boundary.

**Note on Ragtime**: The `ragtime/` directory is licensed under the **Apache License 2.0**. See ragtime/LICENSE for details.

- **Documentation**: docs/
- **Issues**: GitHub Issues
- **Repository**: github.com/thedotmack/claude-mem
- **Official X Account**: @Claude_Memory
- **Official Discord**: Join Discord
- **Author**: Alex Newman (@thedotmack)

**Built with Claude Agent SDK** | **Works with Claude Code** | **Made with TypeScript**

CMEM is a token created by a 3rd party but officially embraced by the creator of Claude-Mem (Alex Newman, @thedotmack). The token acts as a community catalyst for growth and a vehicle for bringing CMEM to the developers and knowledge workers that need it most.

Official BASE CA: 0x76b1967eec0ccaeb001bbbb2b40dc4badba31ba3
