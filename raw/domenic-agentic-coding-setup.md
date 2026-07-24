---
url: https://domenic.me/agentic-coding-setup/
title: My Agentic Coding Setup, July 2026
author: Domenic Denicola
date_fetched: 2026-07-25
date_published: 2026-07-23
---

# My Agentic Coding Setup, July 2026

After leaving corporate work, I had a lot more time to tinker with my AI-assisted development workflow. I've gotten to the point where my setup really has everything I want from day-to-day programming. Since the approaches keep changing quickly, this is hopefully a fun time capsule as much as it is a snapshot of my current preferences and recommendations.

## Requirements

I started using Claude Code in its raw form, from my laptop's terminal app. But I very quickly started to build up a list of requirements:

- The coding agent must run on Linux. All core developers on these projects use macOS, so using Linux uncovered a multitude of hidden bugs (e.g. quoting, special characters, UTF-8 paths, empty file creation).
- Seamless handoff from my desktop to laptop. I don't want to shut down a session when I move from my desk to my couch.
- Work continues even when I close my laptop. Agents need to keep working while I'm away. Sometimes tasks take hours.
- Global settings synced across machines. Everything that's part of my agent's configuration—its system prompt, its preferences, its custom skills—should be up to date everywhere I run agents.
- Review agent work in VS Code. Reading hundreds of lines of agent-generated code in a terminal window is painful.
- Agents need to be able to stand up dev servers. I often ask my agents to "open a preview of this web app." In particular, some of my web development needs access to secure contexts (Service Workers, some web APIs), which often use HTTPS.
- Parallel workstreams without stepping on each others' toes. If I want to get multiple things done at once, my agents should be able to work independently without file conflicts or port collisions, and without having to context switch in a single session.
- Minimal interruption from approval requests. I want to minimize the time I spend manually clicking "yes" on approvals. In Claude Code I aim for --dangerously-skip-permissions; in Codex I aim for --yolo.

## Key Ingredients

### VM + Tailscale

The core of my setup is a quick-to-create and quick-to-replace VM. I created an Ubuntu Server VM on my always-on home desktop machine (using Hyper-V). The VM doesn't need to be too powerful; I gave mine 4 cores and 8 GB RAM.

To connect to the VM I use Tailscale. This connects my desktop, laptop, phone, and VM into a private network. I can SSH in from anywhere. Tailscale SSH is a big quality-of-life improvement (setting the SSH "accept" rule) so I don't need to copy my SSH public key around. I also set up HTTPS certificates and Taildrive.

### Agent Harnesses

Both Claude Code and Codex CLI work great on the VM.

The real winner is the ChatGPT app over SSH. It's truly excellent as a thin client. It handles disconnects and backgrounding very gracefully—a critical feature that many remote development tools get wrong. Copilot doesn't support SSH at all. Claude Code's desktop app has no filesystem-based session grouping, kills sessions when the client closes, and makes you repeatedly refresh its filesystem explorer pane.

### Security (YOLO)

I configure both Claude Code and Codex CLI so they don't ask for any approvals, and my user is in the sudoers file so they can access whatever they need. People have valid concerns about agents deleting your repos, losing secrets, or escaping the VM and hacking into servers on the web. I haven't found that to be an issue in practice—and I run a lot of agents! With aligned models, the main danger is not malice but agentic overaction (e.g., merging a PR I wanted to review before merging).

### Worktrees

Git worktrees let me keep multiple agents running in parallel, each in their own directory with their own branch. I really enjoy that both ChatGPT and Claude desktop apps support a checkbox to create worktrees. However, Claude Code's default location for worktrees is inside the project, and this causes issues with node_modules resolution and recursive commands.

### Code Review in VS Code

I use VS Code's Remote-SSH extension to review my agent's work. This makes it natural to browse code, view diffs, and create review comments in a familiar environment. The ChatGPT app even provides a nice convenient button to open the remote SSH worktree directory, which takes you right into VS Code.

### Dev Servers

My solution to the dev server problem are my tportless wrapper around Portless. tl;dr: Preview URLs of the format `https://agents-base.tail234567.ts.net:8443` are accessible anywhere on the tailnet, have HTTPS (satisfying secure contexts and Service Worker requirements), and each agent's dev server is assigned a unique port, so parallel agents don't interfere.

### Chezmoi

I use chezmoi to sync dotfiles, AGENTS.md, and skills files across my desktop and laptop (and my VM, too). GPT set it up for me, and we keep everything in a private GitHub repo.

## Phone-Based Development

One of the best parts is that developing from my phone falls out almost for free. My typical workflow: I notice a bug when I'm out and about. I open the ChatGPT mobile app on the train, and Tailscale VPN provides the secured connection to my VM. I ask the agent to fix the bug. A few minutes later I get a push notification with a preview URL; I can confirm the fix, open up and merge a pull request, and I've fixed the bug before I even get home.

## Gaps

I wish we had better dev-container infrastructure—I think that would provide better safety and isolation than VMs. I dislike how sessions are strongly tied to the folder path on the VM (renaming a project causes all your session history to disappear). I wish every session transcript got auto-backed up somewhere durable (like a private GitHub repository), both for nostalgia and to help with design record-keeping.

Overall, this is a really fun and exciting time in our industry. I'm sure in six months this entire post will be quaintly obsolete. What a time to be alive!
