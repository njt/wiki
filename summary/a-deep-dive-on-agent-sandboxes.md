---
title: "A Deep Dive on Agent Sandboxes"
url: https://pierce.dev/notes/a-deep-dive-on-agent-sandboxes
author: Pierce Freeman
date_fetched: 2026-05-15
date_published: 2025-09-26
topics:
  - security-and-sandboxing
---

# A Deep Dive on Agent Sandboxes

Pierce Freeman reverse-engineers how Codex CLI sandboxes agent execution, walking through the OS-level primitives across macOS and Linux.

## The Problem

Agents with bash access are dangerous. "You probably wouldn't give your new intern access to the prod credentials. But an arbitrary bash session could certainly provide that permissions escalation."

Existing mitigations:
- **Command whitelisting** (Claude Code, Cursor): "brittle and a bit annoying," "fully impractical to run if you step away from your computer"
- **Full virtualization**: safest but "I'd wager a large sum that almost no one does that"

## Codex's Three Permission Levels

1. **Read Only** — file viewing only; edits, commands, network require approval
2. **Auto (default)** — workspace edits and commands allowed; external/network access requires approval. Uses macOS native sandboxing APIs, not Docker.
3. **Full Access** — unrestricted terminal access

## Execution Pipeline

Every tool call routes through `process_exec_tool_call` based on `SandboxType`. The dispatch starts in `codex-rs/cli/src/main.rs`:

```rust
fn main() -> anyhow::Result<()> {
    arg0_dispatch_or_else(|codex_linux_sandbox_exe| async move {
        cli_main(codex_linux_sandbox_exe).await?;
        Ok(())
    })
}
```

On Linux, if the executable name is `LINUX_SANDBOX_ARG0`, the sandbox subprocess takes over. The routing logic in `codex-rs/core/src/exec.rs`:

```rust
let raw_output_result = match sandbox_type {
    SandboxType::None => exec(params, sandbox_policy, stdout_stream.clone()).await,
    SandboxType::MacosSeatbelt => {
        let child = spawn_command_under_seatbelt(...
```

"By default, you're sandboxed."

## Platform-Specific Implementations

### macOS Seatbelt

Uses Apple's deprecated-but-still-universally-used sandbox subsystem. From `codex-rs/core/src/seatbelt.rs`:

```rust
pub async fn spawn_command_under_seatbelt(
    command: Vec<String>,
    sandbox_policy: &SandboxPolicy,
    cwd: PathBuf,
    ...
) -> std::io::Result<Child> {
    let args = create_seatbelt_command_args(command, sandbox_policy, &cwd);
    env.insert(CODEX_SANDBOX_ENV_VAR.to_string(), "seatbelt".to_string());
    spawn_child_async(
        PathBuf::from(MACOS_PATH_TO_SEATBELT_EXECUTABLE),
        ...
```

Writable roots are enumerated; `.git` directories are carved out as read-only. Network policy is binary:

```rust
let network_policy = if sandbox_policy.has_full_network_access() {
    "(allow network-outbound)\n(allow network-inbound)\n(allow system-socket)"
} else {
    ""
};
```

Seatbelt denies by default, so omitting network permissions suffices.

Three issues with macOS sandboxing:
1. Package management restrictions (e.g., Homebrew)
2. Custom Seatbelt policies are error-prone — a cited bug report: "failed to properly block off access to the user's home directory dotfiles"
3. Limited network control (all-or-nothing)

### Linux Landlock + Seccomp

The `codex-linux-sandbox` helper parses serialized policy, applies restrictions, then exec's the command. From `codex-rs/linux-sandbox/src/linux_run_main.rs`:

```rust
pub fn run_main() -> ! {
    let LandlockCommand { sandbox_policy_cwd, sandbox_policy, command } =
        LandlockCommand::parse();
    if let Err(e) = apply_sandbox_policy_to_current_thread(&sandbox_policy, &sandbox_policy_cwd) {
        panic!("error running landlock: {e:?}");
    }
```

Landlock grants read access everywhere but restricts writes to whitelisted roots (plus `/dev/null`). Rules are applied before `execvp`.

For network isolation via seccomp, specific syscalls are denied:

```rust
deny_syscall(libc::SYS_connect);
deny_syscall(libc::SYS_accept);
deny_syscall(libc::SYS_accept4);
deny_syscall(libc::SYS_bind);
deny_syscall(libc::SYS_listen);
// ... sendto, sendmsg, sendmmsg also denied
// NOTE: allowing recvfrom allows tools like cargo clippy to run
// with their socketpair + child processes for sub-proc management
```

This is "more granular than what you get with Seatbelt on macOS."

## Child Process Management

All sandboxed processes go through `spawn_child_async` in `codex-rs/core/src/spawn.rs`, which:
- Clears the entire environment
- Rebuilds it with only desired variables
- Sets `CODEX_SANDBOX_NETWORK_DISABLED_ENV_VAR=1` when network is blocked
- On Linux, registers a parent-death signal handler via `prctl(PR_SET_PDEATHSIG)` to prevent orphaned processes

## Command Whitelisting

Before execution, commands run through `assess_command_safety` from `codex-rs/core/src/safety.rs`:

```rust
pub fn assess_command_safety(
    command: &[String],
    approval_policy: AskForApproval,
    sandbox_policy: &SandboxPolicy,
    approved: &HashSet<Vec<String>>,
    with_escalated_permissions: bool,
) -> SafetyCheck {
```

Commands are categorized as safe-to-auto-run, needs-approval, or needs-unsandboxed. The trust list is session-scoped. Failed sandboxed commands can be retried unsandboxed with user consent.

## Debugging Support

`codex debug seatbelt` and `codex debug landlock` let users test arbitrary commands through the sandbox, honoring the same `--full-auto` flags as the main CLI.

## Conclusion

Freeman identifies four key design choices:
1. Platform-specific implementations behind a common policy abstraction
2. Default-sandbox execution with selective escalation
3. Session-based trust lists reducing approval fatigue
4. Debug tooling for understanding behavior

"This kind of sandboxing is going to be pretty important" as more people use agents to write and execute code, contrasting with alternatives that essentially plead with the agent not to delete your home directory.
