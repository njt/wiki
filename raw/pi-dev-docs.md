---
url: https://pi.dev/docs/latest
date_fetched: 2026-07-05
backfilled: true
---

## On this page

# Pi Documentation

Pi is a minimal terminal coding harness. It is designed to stay small at the core while being extended through TypeScript extensions, skills, prompt templates, themes, and pi packages.

## Quick start

CopiedInstall Pi with npm:

```
npm install -g --ignore-scripts @earendil-works/pi-coding-agent
```
`--ignore-scripts` disables dependency lifecycle scripts during install. Pi does not require install scripts for normal npm installs.

On Linux or macOS, you can also use the installer:

```
curl -fsSL https://pi.dev/install.sh | sh
```
To uninstall pi itself, use npm for curl and npm installs:

```
npm uninstall -g @earendil-works/pi-coding-agent
```
For pnpm, Yarn, or Bun installs, use the matching global remove command: `pnpm remove -g @earendil-works/pi-coding-agent`, `yarn global remove @earendil-works/pi-coding-agent`, or `bun uninstall -g @earendil-works/pi-coding-agent`.

Then run it in a project directory:

```
pi
```
Authenticate with `/login` for subscription providers, or set an API key such as `ANTHROPIC_API_KEY` before starting pi.

For the full first-run flow, see Quickstart.

## Start here

Copied- Quickstart - install, authenticate, and run a first session.
- Using Pi - interactive mode, slash commands, context files, and CLI reference.
- Providers - subscription and API-key setup for built-in providers.
- Security - project trust, sandbox boundaries, and vulnerability reporting.
- Containerization - sandbox pi with Gondolin, Docker, or OpenShell.
- Settings - global and project settings.
- Keybindings - default shortcuts and custom keybindings.
- Sessions - session management, branching, and tree navigation.
- Compaction - context compaction and branch summarization.

## Customization

Copied- Extensions - TypeScript modules for tools, commands, events, and custom UI.
- Skills - Agent Skills for reusable on-demand capabilities.
- Prompt templates - reusable prompts that expand from slash commands.
- Themes - built-in and custom terminal themes.
- Pi packages - bundle and share extensions, skills, prompts, and themes.
- Custom models - add model entries for supported provider APIs.
- Custom providers - implement custom APIs and OAuth flows.

## Programmatic usage

Copied- SDK - embed pi in Node.js applications.
- RPC mode - integrate over stdin/stdout JSONL.
- JSON event stream mode - print mode with structured events.
- TUI components - build custom terminal UI for extensions.

## Reference

Copied- Session format - JSONL session file format, entry types, and SessionManager API.

## Platform setup

Copied## Development

Copied- Development - local setup, project structure, and debugging.
