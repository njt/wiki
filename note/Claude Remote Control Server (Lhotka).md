# Claude Remote Control Server (Lhotka)

Rockford Lhotka's write-up of graduating from SSH-piloted Claude Code to Claude Code's Remote Control: `/rc` to expose a live session to phone, desktop app, or browser, and `claude remote-control` as a boot-time server that spawns *new* sessions on demand from anywhere — with the Windows scheduled-task mechanics, supervisor script, and idle-detection heuristics needed to make it survive reboots, network loss, and self-updates.

---

The progression is a lesson in what "running agents elsewhere" actually requires. SSH solved the wrong half of the problem: it put Claude on a capable machine but tied the session's life to a TCP connection. `/rc` decouples control from connectivity. `claude remote-control` decouples session *creation* from human presence — after Patch Tuesday wiped every session with a Windows reboot, that became the requirement. The rest of the post is the operational residue of wanting a daemon that Anthropic doesn't ship a service installer for: S4U scheduled tasks, workspace-trust prompts that can't be shown to nobody, hourly `claude update` staging, and idle detection by transcript mtime rather than the misleading `Capacity: 1/32` display.

## Key quotes

> "The session is running on the desktop machine, not inside an SSH connection, so if my laptop goes to sleep or the wifi drops, nothing happens to Claude. It keeps working, and when I reconnect I see whatever it did while I was gone."

The core inversion: the session outlives the client. This is the same conceptual move tmux-plus-mosh made, but relocated into the product's own control plane.

> "With the Remote Control server, the sessions run on my desktop machines, with all my tools... Claude isn't limited by what a generic cloud environment happens to have installed. It has everything I have."

A deliberate rejection of the cloud sandbox in favour of environment continuity. Lhotka's argument is that the value isn't just compute, it's the accumulated per-machine setup — repos, MCP servers, credentials — that no generic environment replicates.

> "The server always keeps one pre-created session, and the number counts sessions that *exist*, not sessions that are *doing something*. What works better is to look at the session transcripts Claude Code writes under `~\.claude\projects`."

A quietly useful observation about observability: the product's own status surface is ambiguous, so the durable ground truth is the filesystem. Fits the broader pattern that agent infrastructure ends up reading raw artifacts rather than trusting dashboards.

> "Every run of the scheduled task is a separate background logon, and one logon isn't allowed to stop a process that belongs to another. So instead of starting the server and exiting, the script stays running as a supervisor for as long as the server runs."

A Windows security boundary (process ownership per logon session) dictates the architecture: not a fire-and-forget launcher but a long-lived supervisor. Classic systems practice wearing agentic clothes.

## Key themes

#tool #pattern #concept

## Analysis

What's strongest here is the honesty about residual problems. Remote Control fixes session death, not the "nobody is logged in" environment: Docker Desktop still needs a login, Git and `gh` still need non-prompting credentials, and the S4U background logon can't touch Credential Manager at all. Anyone reading this as "set up a server, done" misses the point — the second half of the post is an inventory of everything that doesn't follow the session into headlessness. That inventory is the most reusable content in the piece.

The idle-detection design is a genuinely thoughtful compromise: a fixed restart schedule fails for someone crossing time zones, so he requires *two* conditions (staged update + one hour of transcript silence) — and documents the blind spot (a single hour-long silent command can trigger a restart mid-flight). He treats his own heuristic's failure modes as part of the write-up, which is the right register for operational notes.

The marginal contribution to the wiki is a concrete Windows-side data point. Most self-hosted-agent-harness writing targets Linux/containers; Lhotka's scheduled-task-plus-S4U-plus-supervisor pattern, and the specific Credential Manager trap, is the kind of platform-specific knowledge that's expensive to rediscover.

It's also worth noting the post was authored with AI assistance while describing an entirely manual, hand-rolled operations stack — an illustrative asymmetry: the code the agent ecosystem runs on is still maintained the old way, by scripts the human personally supervises.

## Relations

This extends the headless-agent lineage documented in [[Ditch the Token Headache — SSH Just Works]] — Lhotka's earlier SSH setup had the same credential and login-session problems, and this post names which of them survive the switch to Remote Control. It nuances [[Claude Code on the Go]]'s cloud-VM approach by arguing the opposite trade: keep sessions on a fully equipped personal desktop rather than a generic rented environment, betting on environment continuity over clean sandboxes. It complements [[Windows VM Skill for Claude Code]]'s headless-Windows-in-Docker recipe with the native-Windows equivalent (scheduled task, S4U logon, Docker Desktop as the per-user app that won't start). And it deepens [[Cross-Session Messaging]]'s coverage of the same Claude Code control surface from the docs side with a practitioner's operational experience of `/rc` and the remote-control server in production-ish use.

---
*Sources: [[raw/remote-control-server]], [[summary/remote-control-server]]*
*Last updated: 2026-09-23*
