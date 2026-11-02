# SandVault — Sandboxed Agent User Accounts

SandVault (`sv`) is Patrick Wyatt's ~8,500-line Bash/Python toolkit for running AI coding agents and arbitrary shell commands inside a dedicated, locked-down macOS user account — no VM, no container, no Docker. Its thesis is that the oldest isolation primitive on macOS (Unix users) is the right one for YOLO-mode agents: it installs Claude Code, Codex, OpenCode, Gemini, pi and Muse Code wrappers that each run with all permission checks disabled, because the containment boundary is the account plus a `sandbox-exec` profile, not the agent's own approval prompts.

---

## Architecture

The whole system is one 1,959-line Bash script (`sv`) plus satellite scripts:

- **`sv`** — entry point and installer. Builds the `sandvault-<hostuser>` account, writes a sudoers file, generates an SSH ed25519 keypair, lays down ACLs, and on each run composes a `zsh` command string that is either exec'd via `sudo -u` (impersonation mode) or shipped over loopback SSH (`sv --ssh`). The zsh command chains `configure`, sources the user's shell config, optionally starts browser services, then `exec`s the agent.
- **`guest/home/bin/*`** — one wrapper per agent (claude, codex, gemini, opencode, pi, muse). Each detects the Homebrew or native install, then execs the tool in maximum-autonomy mode. They share `sv-tool-prompts.sh`, which injects tool-awareness markdown (browser/CDP and iOS bridge endpoints) via `--append-system-prompt` when those services are running.
- **`helpers/sv-ios-bridge`** — a 648-line Python HTTP server on a dynamic localhost port that whitelists `xcrun simctl` and `iosef` subcommands for a per-session scratch simulator (`sandvault-<uuid>`, deleted on exit). All subprocess calls use explicit argv lists, never a shell.
- **`sv-clone`** — clones a repo into the shared workspace and wires bidirectional git remotes so the host can `git fetch sandvault` work done inside the sandbox.
- **`skills/sandvault/sv/SKILL.md`** — a Claude Code skill implementing a handoff workflow: write a task briefing into the shared workspace, then launch `sv-clone` pointing the sandboxed agent at it.

## Key techniques

- **Three-ACE ACL split.** The shared workspace uses a directory/file ACL split so that new files inherit read/write but not execute: one directory-inherit ACE (search/list, propagates to subdirs), one only-inherit file_inherit ACE, and one non-inheriting per-file ACE. The comments in `sv` explain that this exists because `search`/`list` on a file means execute — a subtle macOS ACL trap.
- **Layered Seatbelt profile** (`/var/sandvault/sandbox-*.sb`, root-owned mode 0444 so the sandbox user can't edit it): default-allow, then deny writes everywhere, deny `/Volumes` reads except the boot disk, deny `file-mount` and the diskarbitrationd/NetAuth/appleeventsd mach services (closing the `osascript`-tells-Finder-to-mount hole), regex-denies on `/dev/r?disk` and `/dev/bpf` as defense in depth beyond POSIX perms, and — the cleverest part — a default-deny on `/Users` reads with last-match-wins re-allows, including literal re-allows on `/Users` and `/Users/Shared` so path traversal itself doesn't fail.
- **Declarative SSH key directory.** `~/.config/codeofhonor/sandvault/authorized_keys.d/` (host-owned, so the untrusted sandbox user can't grant itself access) is the source of truth; every run regenerates the guest's `authorized_keys` in full, making revocation a file delete. It explicitly rejects private keys — a private key pasted there would hand the sandbox a credential — and uses `awk 1` to guarantee newlines so two keys never concatenate.
- **Timeout taxonomy in the iOS bridge.** `SUBPROCESS_TIMEOUT_SEC = 120` default, but `IOSEF_TIMEOUT_SEC = 20` for interactive UI calls and `IOSEF_DESCRIBE_TIMEOUT_SEC = 60` for accessibility-tree reads — the comments document that a hung 120s call outlives the caller's retry loop and makes failure look like an empty response, and that cold-start `describe` setup was being killed mid-flight, causing infinite restarts. A small case study in why one global subprocess timeout is wrong.
- **Re-entrancy via environment.** `SV_SESSION_ID` doubles as a nesting detector (`NESTED=true` disables all setup), a Swift/xcodebuild flag (`--disable-sandbox`, because macOS forbids nested sandboxes), and a hook for user build scripts.
- **Session-scoped children with PID-watch cleanup.** `start-chrome` can't set an EXIT trap (the sourcing zsh `exec`s the agent), so it forks a watcher that polls the session PID and kills Chrome after exit — ignoring HUP so closing the terminal doesn't orphan it.

## Design decisions

- **Account + seatbelt over VM.** Wyatt explicitly tried Docker, Podman, and virtualization first. The trade: far less isolation than a VM (a kernel or privilege-escalation bug in macOS defeats both layers), in exchange for instant context switching, shared filesystem performance, and native tooling (Xcode, Homebrew, Chrome). The threat model is a misbehaving agent, not a motivated attacker with a 0-day.
- **The agent wrapper disables its own sandbox.** Claude Code runs `--dangerously-skip-permissions`, Codex `--dangerously-bypass-approvals-and-sandbox`, Muse Code `--yolo`. The insight: nested approval prompts are friction the OS-level boundary makes redundant, and some agents (Muse) ship sandboxes that would otherwise conflict.
- **Honest about Chrome's `--no-sandbox`.** The README and `start-chrome` both state that macOS won't let Chrome apply its internal sandbox inside a sandboxed process, so a compromised renderer gets the sandvault user's access — no worse than the agent's, but a real weakening versus native Chrome.
- **Shell, all the way down.** No daemons, no config files beyond a sudoers drop-in and a sandbox profile; `sv uninstall` is `rm -rf` of one directory. Homebrew-Cellar installs are handled by rewriting `WORKSPACE` to the stable `opt/` symlink so generated scripts survive upgrades — a detail only a longtime Mac packager would think of.
- **GUI apps are explicitly out of scope**: an app can't report to a WindowServer owned by a different user, and the old `launchctl bsexec` tricks appear closed by modern macOS.

## Comparison notes

- Where [[A Deep Dive on Agent Sandboxes]] surveys container/microVM approaches, SandVault argues the opposite end of the spectrum: OS users plus Seatbelt, trading hard multi-tenancy for zero VM overhead on a personal Mac. Its threat model is closer to [[Fences Not Sandboxes]] — fences for a trusted-ish agent rather than cells for hostile code.
- [[How We Contain Claude]] treats containment as a platform problem; SandVault is the single-developer, zero-infrastructure version of the same idea, notable for how much depth (ACL inheritance rules, mach-service denies, key-revocation directories) fits in Bash.
- Like [[Domenic Denicola's Agentic Coding Setup]], it accepts YOLO agents as a given and moves the safety boundary outside the agent — but does so locally per-machine rather than in disposable cloud VMs.
- The `authorized_keys.d` regenerating directory and bidirectional `sandvault` git remote echo the identity/revoke discipline in [[Session Revocations at Scale]] and the workflow plumbing of [[Git Worktrees]], in miniature.

---
*Sources: [[raw/sandvault]], [[summary/sandvault]]*
*Last updated: 2026-11-02*
