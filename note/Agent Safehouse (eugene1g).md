# Agent Safehouse (eugene1g)

A macOS-native sandboxing wrapper that lets you run coding agents with their own `--yolo` flags while the kernel, not the agent, enforces what's reachable. Deny-by-default file access, auto-granted workdir, single shell script install, shell-function ergonomics so sandboxing is the default you forget about.

---

## What it does

- **Inverts the default.** Agents inherit your full user permissions; Safehouse flips this — "nothing is accessible unless explicitly granted". Read/write on the git root, read on toolchains, everything else denied by the kernel.
- **Kernel-enforced, not prompt-enforced.** `safehouse cat ~/.ssh/id_ed25519` → "Operation not permitted". The block happens before the process sees the data, so no prompt injection can talk its way out.
- **Zero-install ergonomics.** Homebrew or one self-contained Bash script. Shell functions (`claude() { safe claude --dangerously-skip-permissions "$@"; }`) make the sandboxed path the default and `command claude` the explicit escape hatch.
- **LLM-assisted profile generation.** A guided prompt has a model inspect the `sandbox-exec` templates, interview you about your home directory and toolchain, and emit a least-privilege profile — agents writing their own containment.

## Key quotes

> LLMs are probabilistic - 1% chance of disaster makes it a matter of *when*, not *if*.

The framing that justifies the whole tool: YOLO mode isn't a question of whether, but when. Most developers running `--dangerously-skip-permissions` have internalised the risk and run it anyway because permission prompts destroy flow. Safehouse's move is to accept the YOLO desire and redirect it: go ahead, the kernel has you.

> All agents work perfectly in their sandboxes, but can't impact anything outside it.

Not quite true as stated — sandboxes *break* agents regularly (network egress, dotfiles, caches) — but as a design goal it's the right one, and the auto-grants (workdir, toolchains) are the pragmatic concession that makes it mostly hold.

## Themes

#tool #pattern #security

## Analysis

Safehouse is the latest entry in a small, growing genre: single-person, single-script containment for the YOLO workflow. Its distinctive bets are (1) deny-by-default at the kernel rather than an allowlist of things to block, and (2) ergonomics as the actual product — the shell functions mean you never have to remember to sandbox, which is the failure mode of every "run it in the sandbox" policy. The LLM-generates-its-own-least-privilege-profile loop is a genuinely clever inversion: the same model that might delete your home directory is also the best tool for writing the Seatbelt profile that stops it.

The honest caveats: it's macOS-only (`sandbox-exec`/Seatbelt, which Apple has deprecated-but-not-removed), it's filesystem-focused while the scarier agent escapes are network and credential exfiltration, and a single Bash script wrapping Seatbelt is not a security boundary against a determined adversary — it's a seatbelt against your own agent's accidents. That's the right threat model for personal use, and the tool is honest about being one.

## Related

- [[SandVault — Sandboxed Agent User Accounts]] — the closest sibling: also macOS, also YOLO-mode containment for one developer's machine, but via a dedicated user account and root-owned profile rather than Seatbelt wrapping; Safehouse is lighter-weight and more ergonomic, SandVault more isolated.
- [[Sandboxing with Minimal Effort (Inko)]] — makes the same core argument from the tool-builder's side: a security feature's value is its ease of use, not its capability; Safehouse's shell-function ergonomics are that thesis applied to agent containment.
- [[Domenic Denicola's Agentic Coding Setup]] — the disposable-VM answer to the same problem; Safehouse shows the cheaper, kernel-native path for people who won't pay the VM tax, at correspondingly weaker isolation.
- [[nono.sh — Brokered Tool Execution for Agent Security]] — the commercial, policy-and-audit-heavy counterpart: brokered tool calls and Merkle-chained trails versus Safehouse's fire-and-forget kernel deny.

---
*Sources: [[raw/agent-safehouse-dev]], [[summary/agent-safehouse-dev]]*
*Last updated: 2026-11-26*
