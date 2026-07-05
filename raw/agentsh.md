---
url: https://www.agentsh.org/docs/
date_fetched: 2026-07-05
backfilled: true
---

# agentsh Documentation

agentsh ("agent shell") provides execution-layer security for AI agents. Intercept file, network, process, and Postgres-family database activity at runtime—enforce policy, emit audit events, steer actions to approved alternatives.

## Execution-Layer Security (ELS)#

**Execution-Layer Security** is a runtime enforcement model for AI agents. Instead of relying on prompts, tool descriptions, or model alignment to constrain behavior, ELS intercepts the actual system calls—file I/O, network connections, process spawning, signals—and enforces policy at the kernel level. The agent never sees the enforcement; it simply cannot perform operations that policy does not permit.

agentsh is the reference implementation of ELS. It sits *under* your agent/tooling—intercepting **file**, **network**, **process**, **database**, and **signal** activity (including subprocess trees), enforcing the policy you define, and emitting **structured audit events**. Learn more about execution-layer security →

#### Drop-in Shell

Turns every command (and its subprocesses) into auditable events.

#### Policy Engine

Per-operation decisions: allow, deny, approve, redirect, or soft_delete.

#### Full Visibility

File I/O, network, DNS, process lifecycle, PTY activity, and LLM requests.

#### Two Output Modes

Human-friendly shell output or compact JSON for agents/tools.

#### Redirect & Steer

Swap commands or file paths to keep agents on the paved road.

#### LLM Proxy & DLP

Intercept API requests, redact PII, and track token usage.

#### Database Proxy

Enforce connection and statement policy on declared Postgres services.

#### Checkpoints

Snapshot workspace state and rollback destructive operations.

#### Cross-Platform

Linux, macOS, and Windows with native enforcement mechanisms.

### Why agentsh?

Agent workflows eventually run arbitrary code (`pip install`, `make test`, `python script.py`). Traditional "ask for approval before running a command" controls stop at the tool boundary and can't see what happens *inside* that command.

agentsh enforces policy **at runtime**, so hidden work done by subprocesses is still governed, logged, and (when required) approved.

## Quick Start#

Run a command that spawns subprocesses and touches the filesystem and network to see agentsh in action.

```
# 1) Create a session in your repo/workspace
SID=$(agentsh session create --workspace . --json | jq -r .id)
# 2) Run something simple (human-friendly output)
agentsh exec "$SID" -- uname -a
# 3) Run something that hits the network (JSON output + event summary)
agentsh exec --output json --events summary "$SID" -- curl -s https://example.com
# 4) Trigger a policy decision - try to delete something
agentsh exec "$SID" -- rm -rf ./tmp
# 5) See what happened (structured audit trail)
agentsh exec --output json --events all "$SID" -- ls
```
**What you'll see in the JSON output:**

- `exit_code`: the command's exit status
- `stdout`/- `stderr`: captured output
- `events[]`: every file/network/process operation with policy decisions
- `policy.decision`:- `allow`,- `deny`,- `approve`, or- `redirect`

## Installation#

### From a GitHub Release

Download the `.deb`, `.rpm`, Arch package, or tarball for your platform from the releases page.

```
# Example for Debian/Ubuntu
sudo dpkg -i agentsh_<VERSION>_linux_amd64.deb
```
### From Source (Linux)

```
make build
sudo install -m 0755 bin/agentsh bin/agentsh-shell-shim /usr/local/bin
```
### From Source (macOS)

```
# Requires Xcode 15+
make build-macos-enterprise
```
## Security Modes#

agentsh selects the best configured security mode based on available kernel features. Set `security.mode: auto` (the default) and agentsh detects available primitives at startup. Ptrace enforcement is an opt-in backend configured under `sandbox.ptrace`, not a `security.mode` value.

| Mode | Requirements | Protection | Description | 
|---|---|---|---|
| `full` | seccomp + eBPF + FUSE | 100% | Full security with all features | 
| `landlock` | Landlock + FUSE | ~85% | Kernel-enforced execution and FS control | 
| `landlock-only` | Landlock | ~80% | Landlock without FUSE granularity | 
| `minimal` | (none) | ~50% | Capability dropping and shim policy only | 

Run `agentsh detect` to see a weighted protection score (0–100) grouped by domain—File Protection, Command Control, Network, Resource Limits, and Isolation—with actionable tips for missing capabilities. See Detecting Capabilities.

#### Security configuration

```
security:
  mode: auto              # auto | full | landlock | landlock-only | minimal
  strict: false           # Fail if mode requirements not met
  minimum_mode: ""        # Fail if auto-detect picks worse than this
  warn_degraded: true     # Log warnings when running in degraded mode
```
#### Landlock configuration (Linux 5.13+)

When Landlock is available, configure filesystem and network restrictions:

```
landlock:
  enabled: true
  # Directories where command execution is allowed
  allow_execute:
    - /usr/bin
    - /bin
    - /usr/local/bin
  # Directories where reading is allowed
  allow_read:
    - /etc/ssl/certs
    - /lib
    - /usr/lib
  # Paths explicitly denied (container escape vectors)
  deny_paths:
    - /var/run/docker.sock
    - /run/containerd/containerd.sock
    - /var/run/secrets/kubernetes.io
  # Network restrictions (requires kernel 6.7+ / Landlock ABI v4)
  network:
    allow_connect_tcp: true   # Allow outbound TCP
    allow_bind_tcp: false     # Allow listening on TCP ports
```
#### Feature matrix by mode

| Feature | full | landlock | landlock-only | ptrace | minimal | 
|---|---|---|---|---|---|
| Execution control (shim) | Yes | Yes | Yes | Yes | Yes | 
| Execution control (kernel) | seccomp | Landlock | Landlock | ptrace | No | 
| Filesystem (fine-grained) | FUSE | FUSE | Landlock | ptrace | No | 
| File enforcement (no FUSE) | seccomp-notify | Landlock | Landlock | ptrace | No | 
| Signal interception | seccomp | CAP_KILL* | CAP_KILL* | ptrace | CAP_KILL* | 
| Network (kernel) | eBPF | Landlock** | Landlock** | ptrace | No | 
| Execve interception | seccomp | No | No | ptrace | No | 

*Relies on dropped CAP_KILL to prevent signaling external processes. **Requires kernel 6.7+ (Landlock ABI v4).

## Scaling with Canyon Road#

agentsh works standalone. When you need fleet-wide visibility and central management, it integrates with the Canyon Road platform:

- **Beacon**— Monitors AI desktop tools (Claude, Cursor, ChatGPT) on your endpoints. Different from agentsh: Beacon watches what apps do at the OS level, while agentsh wraps agents you run directly.
- **Watchtower**— The control plane for both. Centralized policies, approval routing (Slack, email, SMS), SIEM export, and a kill switch when things go wrong.

Learn more at canyonroad.ai/how-it-works.

See the GitHub repository for more examples, the full spec, and to report issues.
