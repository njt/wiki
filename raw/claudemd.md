---
url: http://claude.md
date_fetched: 2026-07-05
backfilled: true
---

## Get started

Choose your environment to get started. Most surfaces require a Claude subscription or Anthropic Console account. The Terminal CLI and VS Code also support third-party providers.- Terminal
- VS Code
- Desktop app
- Web
- JetBrains

The full-featured CLI for working with Claude Code directly in your terminal. Edit files, run commands, and manage your entire project from the command line.To install Claude Code, use one of the following methods:If you see You can also install with apt, dnf, or apk on Debian, Fedora, RHEL, and Alpine.Then start Claude Code in any project:You’ll be prompted to log in on first use. That’s it! Continue with the Quickstart →

- Native Install (Recommended)
- Homebrew
- WinGet

**macOS, Linux, WSL:**

**Windows PowerShell:**

**Windows CMD:**

`The token '&&' is not a valid statement separator`, you’re in PowerShell, not CMD. If you see `'irm' is not recognized as an internal or external command`, you’re in CMD, not PowerShell. Your prompt shows `PS C:\` when you’re in PowerShell and `C:\` without the `PS` when you’re in CMD.If the install command fails with `syntax error near unexpected token '<'`, a `403`, or another curl error, see Troubleshoot installation to match the error to a fix and for alternative install methods.Git for Windows is recommended on native Windows so Claude Code can use the Bash tool. If Git for Windows is not installed, Claude Code uses PowerShell as the shell tool instead. WSL setups do not need Git for Windows.Native installations automatically update in the background to keep you on the latest version.

## What you can do

Here are some of the ways you can use Claude Code:Automate the work you keep putting off


Automate the work you keep putting off

Claude Code handles the tedious tasks that eat up your day: writing tests for untested code, fixing lint errors across a project, resolving merge conflicts, updating dependencies, and writing release notes.

Build features and fix bugs


Build features and fix bugs

Describe what you want in plain language. Claude Code plans the approach, writes the code across multiple files, and verifies it works.For bugs, paste an error message or describe the symptom. Claude Code traces the issue through your codebase, identifies the root cause, and implements a fix. See common workflows for more examples.

Create commits and pull requests


Create commits and pull requests

Claude Code works directly with git. It stages changes, writes commit messages, creates branches, and opens pull requests.In CI, you can automate code review and issue triage with GitHub Actions or GitLab CI/CD.

Connect your tools with MCP


Connect your tools with MCP

The Model Context Protocol (MCP) is an open standard for connecting AI tools to external data sources. With MCP, Claude Code can read your design docs in Google Drive, update tickets in Jira, pull data from Slack, or use your own custom tooling. The MCP quickstart connects your first server end to end.

Customize with instructions, skills, and hooks


Customize with instructions, skills, and hooks

`CLAUDE.md` is a markdown file you add to your project root that Claude Code reads at the start of every session. Use it to set coding standards, architecture decisions, preferred libraries, and review checklists. Claude also builds auto memory as it works, saving learnings like build commands and debugging insights across sessions without you writing anything.Create skills to package repeatable workflows your team can share, like `/review-pr` or `/deploy-staging`.Hooks let you run shell commands before or after Claude Code actions, like auto-formatting after every file edit or running lint before a commit.Run agent teams and build custom agents


Run agent teams and build custom agents

Spawn multiple Claude Code agents that work on different parts of a task simultaneously. A lead agent coordinates the work, assigns subtasks, and merges results.To run several full sessions in parallel and watch them from one screen, use background agents. For fully custom workflows, the Agent SDK lets you build your own agents powered by Claude Code’s tools and capabilities, with full control over orchestration, tool access, and permissions.

Pipe, script, and automate with the CLI


Pipe, script, and automate with the CLI

Claude Code is composable and follows the Unix philosophy. Pipe logs into it, run it in CI, or chain it with other tools:See the CLI reference for the full set of commands and flags.

Schedule recurring tasks


Schedule recurring tasks

Run Claude on a schedule to automate work that repeats: morning PR reviews, overnight CI failure analysis, weekly dependency audits, or syncing docs after PRs merge.

- Routines run on Anthropic-managed infrastructure, so they keep running even when your computer is off. They can also trigger on API calls or GitHub events. Create them from the web, the Desktop app, or by running `/schedule`in the CLI.
- Desktop scheduled tasks run on your machine, with direct access to your local files and tools
- `/loop`repeats a prompt within a CLI session for quick polling

Work from anywhere


Work from anywhere

Sessions aren’t tied to a single surface. Move work between environments as your context changes:

- Step away from your desk and keep working from your phone or any browser with Remote Control
- Message Dispatch a task from your phone and open the Desktop session it creates
- Kick off a long-running task on the web or iOS app, then pull it into your terminal with `claude --teleport`. Teleport requires a claude.ai subscription.
- Hand off a terminal session to the Desktop app with `/desktop`for visual diff review
- Route tasks from team chat: mention `@Claude`in Slack with a bug report and get a pull request back

## Use Claude Code everywhere

Each surface connects to the same underlying Claude Code engine, so your CLAUDE.md files, settings, and MCP servers work across all of them. Beyond the Terminal, VS Code, JetBrains, Desktop, and Web environments above, Claude Code integrates with CI/CD, chat, and browser workflows:| I want to… | Best option | 
|---|---|
| Continue a local session from my phone or another device | Remote Control | 
| Push events from Telegram, Discord, iMessage, or my own webhooks into a session | Channels | 
| Start a task locally, continue on mobile | Web or Claude iOS app | 
| Run Claude on a recurring schedule | Routines or Desktop scheduled tasks | 
| Automate PR reviews and issue triage | GitHub Actions or GitLab CI/CD | 
| Get automatic code review on every PR | GitHub Code Review | 
| Route bug reports from Slack to pull requests | Slack | 
| Debug live web applications | Chrome | 
| Build custom agents for your own workflows | Agent SDK | 

## Next steps

Once you’ve installed Claude Code, these guides help you go deeper.- Quickstart: walk through your first real task, from exploring a codebase to committing a fix
- Store instructions and memories: give Claude persistent instructions with CLAUDE.md files and auto memory
- Common workflows and best practices: patterns for getting the most out of Claude Code
- Settings: customize Claude Code for your workflow
- Troubleshooting: solutions for common issues
- code.claude.com: demos, pricing, and product details
