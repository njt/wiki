---
title: "A Deep Dive on Agent Sandboxes"
url: https://pierce.dev/notes/a-deep-dive-on-agent-sandboxes
date_fetched: 2026-05-14
section: "LLMs"
---

# A Deep Dive on Agent Sandboxes

## Main Topic
Pierce Freeman examines how coding agents like Claude can be safely executed through sandboxing technologies, analyzing Codex's implementation across macOS and Linux platforms.

## Key Arguments

**The Core Problem:**
Agents with bash access are powerful but dangerous. "You probably wouldn't give your new intern access to the prod credentials. But an arbitrary bash session could certainly provide that permissions escalation."

**Existing Solutions and Their Limitations:**
- Command whitelists (used in Claude Code and Cursor) require constant human approval, making them impractical for unattended execution
- Full virtualization is safest but rarely implemented in practice

**Codex's Three Permission Levels:**
1. Read Only - restricted to file viewing
2. Auto (default) - workspace edits and commands allowed, but network and external access blocked
3. Full Access - unrestricted permissions

## Technical Implementation

**Platform-Specific Approaches:**

*macOS Seatbelt:*
- Uses Apple's native sandboxing framework
- Restricts writable directories while keeping `.git` read-only
- Simple binary network access control

*Linux (Landlock + seccomp):*
- Landlock handles filesystem restrictions
- Seccomp-BPF filters system calls to block network operations
- More granular than macOS alternatives
- Separate `codex-linux-sandbox` subprocess applies policies before command execution

**Design Principles:**
- Sandboxing is default, not optional
- Session-scoped trust lists reduce approval fatigue
- Failed sandboxed commands can escalate with user consent
- Environment variables are completely cleared then selectively rebuilt

## Notable Limitations

The article identifies OS-level sandbox constraints:
- Coarse-grained network control (all-or-nothing rather than domain-specific blocking)
- Package management complications on macOS
- Policy complexity creating security risks through implementation mistakes

## Conclusions

Freeman argues this approach balances practical usability with security, offering "platform-specific implementations unified behind a common policy abstraction." As agent usage grows, such sandboxing becomes essential infrastructure rather than optional hardening.
