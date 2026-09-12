# Cross-Session Messaging

Anthropic's official documentation for Claude Code's cross-session messaging (v2.1.224+ on macOS/Linux/WSL 2, v2.1.234+ on native Windows): independent sessions discover each other with `ListAgents` and exchange plain-text messages with `SendMessage`, over per-session inbox sockets on the same machine or via Remote Control across machines. The feature ships on-by-default, and the doc's real substance is the delivery contract — delivered/held/refused, inbound controls computed from the two sessions' permission-mode classes, and the rule that a peer message is never the user's consent.

---

## Key Quotes

> "When a session meets the requirements, messaging is on with nothing to enable."

On by default. Unlike MCP servers or hooks, this channel isn't something you opt into — the coordination fabric is just there, which is itself a statement: multi-session operation is being treated as the normal case, not the power-user case.

> "It can't approve anything: a message from another session never counts as your consent, so it can't answer a pending permission prompt on your behalf."

The security spine of the design. A peer session is not the user. The doc extends the rule: peer messages can't change permission settings or `CLAUDE.md`, and commands in message text arrive as plain text, never executed. Peer input is untrusted input — the prompt-injection posture, applied to agents as senders.

> "Claude Code delivers these messages over a per-session socket on your machine, never through Anthropic servers."

Same-machine transport is local-first: a Unix domain socket (named pipe on Windows), registered on disk, with safety checks on the reply target — symlinked targets and wrong-process endpoints are refused. Cross-machine is where locality breaks: those sends route through Remote Control and need claude.ai sign-in, and enterprise providers (Amazon Bedrock, Claude Platform on AWS, Google Cloud's Agent Platform, Microsoft Foundry) get same-machine messaging only, from v2.1.248.

> "A message loop between two sessions therefore stops on its own."

The limits section is the best part of the doc: a ~1M-character size cap, burst refusal at the sender, per-sender rate limits, identical-repeat dropping, and a 50-message queue cap in the receiving session. Someone thought about what happens when two agents discover they can talk to each other, and the answer is throttling rather than prohibition.

> "Claude can decide to send a message without being asked, and you can also prompt for one."

The quiet headline. The coordination boundary moves from your copy-paste between terminals to the model's judgment — sessions handing findings, landings, and status to each other is now the agent's call, not just yours.

---

## How It Works

**Discovery and addressing.** `/list-agents` (also `/peers`) shows subagents, teammates, local sessions (only those that bound an inbox socket), and cloud/Remote Control sessions. Sessions answer to names set via `/rename` or `--name`; collisions get disambiguated with short identifiers, and renaming updates a shared on-disk record — with a warning when it can't.

**Delivery states.** Every message lands in one of three states: **Delivered**, **Held** (until a mode or settings change, or your approval, allows it), or **Refused** (dropped, reason reported back). `crossSessionInbound` sets policy per session; with no value applied, delivery is computed per message from the two sessions' **permission-mode classes** — bypass-prompting sessions in one class, everyone else in the other. A prompting receiver holds messages from bypassing senders; a bypassing receiver holds everything except messages from other bypassing senders.

**Idle notices.** `notify_when_idle` subscribes to one notice when a watched local session next goes idle or exits — one-shot, "neither session polls the other," 12-hour TTL, and refused for teammates, subagents, or cross-machine targets. A migration or test run reports back instead of being polled.

**The inbox socket.** Each session binds a socket, exported to hooks and Bash as `CLAUDE_CODE_MESSAGING_SOCKET` with a per-session `CLAUDE_CODE_MESSAGING_TOKEN` (auth line required on native Windows, optional elsewhere). Scripts can post into their own session; own-child messages are verified by process evidence or the token, and sandboxed commands need explicit Unix-socket settings to reach the socket. `claude -p` workers bind inboxes like interactive sessions; bare mode doesn't. For an unattended `-p` worker, set `crossSessionInbound: accept` in `--settings` — otherwise held messages expire at `dialogExpiry` (five minutes by default).

**Topology boundaries.** Reachability is filesystem visibility: containers are islands, WSL 2 and native Windows can't reach each other, and sessions past a bounded page of cloud/remote listings can't be messaged by name at all.

## Themes

#tool #concept #pattern

- **Agent-to-agent transport** — a first-party message bus between independent top-level sessions, deliberately distinct from subagents (inside one session) and agent teams (one supervised unit).
- **Permission-mode classes as trust levels** — delivery policy computed from whether a session bypasses prompts: a coarse but machine-checkable proxy for trust.
- **Inbox as attack surface** — auth tokens, process-evidence checks, symlink/endpoint safety checks, sandbox socket controls. The inbox gets the same defensive seriousness as tool permissions.
- **Loop-safety by throttling** — burst refusal, repeat-dropping, queue caps: coordination failure modes handled inside the channel itself.

---

## Analysis

**This is a message bus with a security model, not a chat feature.** The interesting design decision is that trust is anchored to permission-mode *classes* rather than identity. There's no key, no handshake, no ACL — delivery policy falls out of a property the harness already tracks. That's elegant, but coarse: a `bypassPermissions` session is trusted to talk to another bypassing session regardless of what either is doing, and plan mode counts as bypassing in sessions where bypass is available — a subtlety that will surprise someone. The peer-never-approves rule keeps the human as the only trust anchor that crosses classes, which is the right invariant.

**The local/remote split is the honest part.** Same-machine messaging is genuinely peer-to-peer: sockets on disk, no Anthropic servers, works on every provider. Cross-machine messaging routes through Remote Control, can't be reached with an API key, and older sessions fall off the back of a bounded listing. The doc doesn't oversell it: offline targets get store-and-forward, and sends without Remote Control arrive with no reply address. Read as an architecture statement: same-machine coordination is solved; the cross-machine part is rented.

**It officializes a workflow people already run by hand.** The non-interactive section is written for the overnight-daemon pattern — a headless `claude -p` worker that can receive a message mid-run, with explicit rules for what happens to held messages when no terminal is attached. Between this, channels for external events, and agent view for steering many sessions, the copy-paste-between-terminals era of multi-session work has a first-party replacement. What it doesn't do is supervise: no queue, no retry, no scheduler — that remains orchestrator territory, and this channel gives those tools a native transport to build on.

**Version churn is the tell.** The doc is gated on at least nine version pins (v2.1.224 through v2.1.251), with behavior changes called out at each — previews replaced full-text display at v2.1.247, burst reporting was silently wrong before v2.1.236. The feature is moving fast enough that the page reads as a changelog wearing a reference doc's clothes. Treat any specific behavior here as a snapshot of September 2026.

## Related Pages

- [[All Your Agents Are Going Async]] argues async agents lack durable transport and that a session "should be a thing that humans and agents can connect to, and disconnect from at any time." This doc is the vendor shipping a piece of that — held messages, idle notices, and inbox sockets that survive terminal detach are durable-transport semantics — but only within Claude Code's own sessions, which nuances "nobody has both" into "the vertical stacks are building it for themselves first."
- [[Swarm Skill]]'s hardest rules (serialize sibling communication, verify agent health) exist because fan-outs inside one session had no peer channel; this feature gives independent top-level sessions what swarms lack, and its burst refusal and repeat-dropping codify the swarm skill's "bursts kill" lesson into the transport layer itself.
- [[Automating Myself Out of Development]]'s cron-driven `claude -p` daemon uses GitHub issues as its only channel and re-queries work to check status; the `-p` inbox support here (`crossSessionInbound: accept`, one-shot idle notices) is the native transport that workflow was missing — the daemon could be messaged mid-run instead of polled.
- [[Fleet Supervisor (sermakarevich)]] is an external control plane doing with beads-queues and Telegram what this channel does natively; it strengthens the case for supervisors (someone still has to schedule, retry, and reap) while obsoleting their hand-rolled status polling, and it complicates the build-vs-buy calculus for anyone building a fleet tool around Claude Code.

---
*Sources: [[raw/cross-session-messaging]], [[summary/cross-session-messaging]]*
*Last updated: 2026-09-13*
