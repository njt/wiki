# Windows VM Skill for Claude Code

Jesse Vincent's (obra) raw Claude Code command definition for a skill named `windows-vm`: create, start, stop, restart, or SSH into a headless Windows 11 VM running in Docker with KVM acceleration. The file is both artifact and argument — it shows what a "skill as runbook" looks like when the runbook has to survive Windows-specific hostility (sshd PATH propagation, execution policies, broken winget, rotated host keys) and encodes all of it as executable procedure rather than advice. It is the primary source behind the companion write-up [[Windows in Docker]].

---

## What it is

The frontmatter is the Claude Code command format in miniature: a `name` (`windows-vm`), a `description` written as a trigger condition for the model, an `argument-hint` (`[create|start|stop|restart|ssh|status]`), and `allowed-tools: Bash, Read, Write`. Everything below it is the procedure: prerequisites (Docker, `/dev/kvm`, `sshpass`, optional ImageMagick), the on-disk layout, the full `docker run` invocations for both the cached-ISO and first-boot cases, the post-install PowerShell script piped over SSH, and a gotchas list that reads like a debugging diary.

The architecture is deliberately austere: no RDP, no GUI, no display at any point in the happy path — SSH on `localhost:2222` is the only interface, with a VNC-in-browser console on 8006 reserved for debugging and RDP on 3389 as a fallback. Every port is bound to 127.0.0.1 only; other machines reach it by jumping through the Tailscale host (`ssh -J jesse@magic-kingdom -p 2222 user@localhost`).

## Key Quotes

> "Create, manage, or connect to a headless Windows 11 VM running in Docker with SSH access. Use when the user wants to spin up, stop, restart, or SSH into a Windows VM."

The description is written for the router, not the reader — it tells the model *when* to load this skill, which is the load-bearing part of the format. The human-facing documentation lives elsewhere (the blog post); the skill file only needs to fire correctly.

> "Node.js is not pre-installed — the Claude Code install script (`irm https://claude.ai/install.ps1 | iex`) will report success but `claude` won't work without Node. Install Node.js via MSI first."

A trap that generates a *false success signal* and then a confusing failure. This is exactly the kind of knowledge that vanishes from chat transcripts but compounds inside a skill file — the skill exists because someone already paid for these lessons.

> "Must add it to the **system** PATH (not user PATH) because OpenSSH's sshd only reads system PATH."

And its sibling: "Interactive SSH sessions don't get full PATH — Windows OpenSSH sshd doesn't properly propagate the system PATH to interactive PowerShell sessions. Fix: create a system-wide PowerShell profile (`$PROFILE.AllUsersAllHosts`) that rebuilds `$env:Path` from the registry on every login." Two different PATH failures with two different fixes, both Windows-specific, both invisible until you've SSH'd in and watched `claude` not exist.

> "Ports are bound to `127.0.0.1` only — not exposed to the network. Access from other machines via Tailscale SSH tunneling."

The security posture is default-deny at the network layer and throwaway at the credential layer (`user`/`password` on a loopback-only service is a calculated risk, not an oversight). The VM is disposable; the perimeter is the tailnet.

> "Escaping hell — Running PowerShell commands over SSH with nested quotes is unreliable. Pipe scripts via stdin using `powershell -ExecutionPolicy Bypass -Command -` instead."

The bash→SSH→cmd→PowerShell quoting stack defeats inline quoting entirely; the stdin/heredoc pattern is the only reliable channel. Note the pragmatic irony of `Bypass` inside a document that elsewhere carefully sets `RemoteSigned` at `LocalMachine` scope — policy is loosened exactly where it blocks automation and tightened where it persists.

## Key Themes

#tool #concept

- **Skill as runbook.** The file doesn't describe how to manage a VM; it *is* the procedure, with decision branches (cached ISO vs. first boot) written into the body. Compare [[Claude Code Skills System]]'s description of the mechanism — this is a live specimen of the command format doing real infrastructure work.
- **VM as agent runtime.** The end state of a 20–30 minute fresh install is one thing: `claude --version` answering over SSH. The Windows box exists solely so Claude Code has somewhere to run — disposable compute for agent work, with warm boots at ~2 minutes making the disposability economical.
- **Caching discipline enables disposability.** The single most important design decision is storing the 7.3GB ISO *outside* the dockur-managed `/storage` volume, which is wiped on recreate, and mounting it back as `/boot.iso`. Disposable infrastructure only pays off when the expensive, immutable parts never live in the disposable layer.
- **Windows as the awkward tier.** Every gotcha is Windows-specific arcana: sshd PATH semantics, PowerShell execution scopes, winget certificate failures, OOBE-timed OEM scripts. Cross-platform agent infrastructure is only as portable as its worst-supported OS.

## Critical Analysis

The honest read is that this file is valuable precisely because it is unglamorous. There is no cleverness here — just a sequence of commands with the failure modes pre-absorbed. That is the point of the skill format working as intended: the *second* time Jesse needs a Windows VM, the 20–30 minutes of OOBE-plus-OpenSSH waiting and the half-day of PATH debugging compress into one argument. The mangled markdown of the gist (tables collapsed into pipe soup, lists run together) suggests the primary consumer is the model, not a human — a document optimized for execution rather than reading.

Two criticisms. First, plaintext credentials (`user`/`password`) and a real git identity are baked into the command body — acceptable on a personal tailnet with loopback-only exposure, but the file is on a public gist, and the pattern it teaches (secrets in skill files) doesn't survive being copied into shared repos. Second, the procedure is imperative shell all the way down; nothing here is idempotent in the configuration-management sense, and "remove the old container and disk" is the recreate story. A terraform-or-bust engineer would wince — but for a single-user, single-host, occasionally-recreated VM, a runbook beats a dependency graph.

Positioned against the wiki: [[Windows in Docker]] is the narrative derived from this artifact (written by Claude Opus 4.6 at Jesse's request, per that page), so this ingest supplies the primary source that page summarizes. It nuances [[Domenic Denicola's Agentic Coding Setup]] — same disposable-VM-plus-Tailscale pattern, one OS over, paying a distinctly Windows tax Denicola never encounters on Ubuntu. And it complicates [[A Deep Dive on Agent Sandboxes]]: the sandbox-platform convergence story (Firecracker, microVMs) has no answer for "the agent needs to run Windows," where full QEMU emulation in Docker remains the only practical route.

---

*Sources: [[raw/windows-vm-claude-code-command]], [[summary/windows-vm-claude-code-command]]*
*Last updated: 2026-09-22*
