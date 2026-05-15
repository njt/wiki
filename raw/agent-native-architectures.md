---
url: https://every.to/guides/agent-native
title: Agent-native Architectures
author: Dan Shipper / Every
date_fetched: 2026-05-15
date_published: 2026-01-17
---

A technical guide for building applications where agents are first-class citizens.

# Agent-native Architectures

## Why now

Software agents work reliably now. Claude Code demonstrated that a large language model (LLM) with access to bash and file tools, operating in a loop until an objective is achieved, can accomplish complex multi-step tasks autonomously.

The surprising discovery: A really good coding agent is actually a really good general-purpose agent. The same architecture that lets Claude Code refactor a codebase can let an agent organize your files, manage your reading list, or automate your workflows.

The Claude Code software development kit (SDK) makes this accessible. You can build applications where features aren't code you write—they're outcomes you describe, achieved by an agent with tools, operating in a loop until the outcome is reached.

This opens up a new field: software that works the way Claude Code works, applied to categories far beyond coding.

## Core principles

### 1. Parity

Whatever the user can do through the UI, the agent should be able to achieve through tools. This is the foundational principle. Without it, nothing else matters.

Ensure the agent has tools that can accomplish anything the UI can do.

The test: Pick any UI action. Can the agent accomplish it?

### 2. Granularity

Tools should be atomic primitives. Features are outcomes achieved by an agent operating in a loop.

A tool is a primitive capability. A feature is an outcome described in a prompt, achieved by an agent with tools, operating in a loop until the outcome is reached.

The test: To change behavior, do you edit prompts or refactor code?

### 3. Composability

With atomic tools and parity, you can create new features just by writing new prompts.

Want a "weekly review" feature? That's just a prompt: "Review files modified this week. Summarize key changes. Based on incomplete items and approaching deadlines, suggest three priorities for next week." The agent uses list_files, read_file, and its judgment. You described an outcome; the agent loops until it's achieved.

### 4. Emergent capability

The agent can accomplish things you didn't explicitly design for.

The flywheel:
1. Build with atomic tools and parity
2. Users ask for things you didn't anticipate
3. Agent composes tools to accomplish them (or fails, revealing a gap)
4. You observe patterns in what's being requested
5. Add domain tools or prompts to make common patterns efficient
6. Repeat

The test: Can it handle open-ended requests in your domain?

### 5. Improvement over time

Agent-native applications get better through accumulated context and prompt refinement. Unlike traditional software, agent-native applications can improve without shipping code.

- Accumulated context: State persists across sessions via context files
- Developer-level refinement: Ship updated prompts for all users
- User-level customization: Users modify prompts for their workflow

## Principles in practice

### Parity in practice

A capability map helps:

| User Action | How Agent Achieves It |
|---|---|
| Create a note | write_file to notes directory, or create_note tool |
| Tag a note as urgent | update_file metadata, or tag_note tool |
| Search notes | search_files or search_notes tool |
| Delete a note | delete_file or delete_note tool |

### Granularity: Less vs. more

**Less granular**: Tool: classify_and_organize_files(files) → You wrote the decision logic → Agent executes your code → To change behavior, you refactor. Bundles judgment into the tool. Limits flexibility.

**More granular**: Tools: read_file, write_file, move_file, bash. Prompt: "Organize the downloads folder..." → Agent makes the decisions → To change behavior, edit the prompt. Agent pursues outcomes with judgment. Empowers flexibility.

### From primitives to domain tools

Start with pure primitives: bash, file operations, basic storage. This proves the architecture works and reveals what the agent actually needs.

As patterns emerge, add domain-specific tools deliberately. Use them to:
- **Anchor vocabulary**: A create_note tool teaches the agent what "note" means in your system
- **Add guardrails**: Some operations need validation that shouldn't be left to agent judgment
- **Improve efficiency**: Common operations can be bundled for speed and cost

The rule for domain tools: They should represent one conceptual action from the user's perspective. They can include mechanical validation, but judgment about what to do or whether to do it belongs in the prompt.

Keep primitives available. Domain tools are shortcuts, not gates. Unless there's a specific reason to restrict access (security, data integrity), the agent should still be able to use underlying primitives for edge cases. This preserves composability and emergent capability.

The default is open; make gating a conscious decision.

### Graduating to code

Some operations will need to move from agent-orchestrated to optimized code for performance or reliability:

1. Agent uses primitives in a loop — Flexible, proves the concept
2. Add domain tools for common operations — Faster, still agent-orchestrated
3. For hot paths, implement in optimized code — Fast, deterministic

The caveat: Even when an operation graduates to code, the agent should be able to trigger the optimized operation itself and fall back to primitives for edge cases the optimized path doesn't handle. Graduation is about efficiency. Parity still holds.

## Files as the universal interface

Agents are naturally good at files. Claude Code works because bash + filesystem is the most battle-tested agent interface.

- **Already Known**: Agents already know cat, grep, mv, mkdir. File operations are the primitives they're most fluent with.
- **Inspectable**: Users can see what the agent created, edit it, move it, delete it. No black box.
- **Portable**: Export is trivial. Backup is trivial. Users own their data.
- **Syncs Across Devices**: On mobile with iCloud, all devices share the same file system. Agent's work appears everywhere—without building a server.
- **Self-Documenting**: /projects/acme/notes/ is self-documenting in a way that SELECT * FROM notes WHERE project_id = 123 isn't.

A general principle of agent-native design: Design for what agents can reason about. The best proxy for that is what would make sense to a human. If a human can look at your file structure and understand what's going on, an agent probably can too.

### Directory structure conventions

Entity-scoped directories: `{entity_type}/{entity_id}/` containing primary content, metadata, and related materials.

Example: `Research/books/{bookId}/` contains full text, notes, sources, and agent logs.

File naming patterns:
- Entity data: `{entity}.json` — library.json, status.json
- Human-readable content: `{content_type}.md` — introduction.md, profile.md
- Agent reasoning: `agent_log.md` — Per-entity agent history
- Primary content: `full_text.txt` — Downloaded/extracted text
- Checkpoints: `{sessionId}.checkpoint` — UUID-based
- Configuration: `config.json` — Feature settings

### The context.md pattern

```
# Context
## Who I Am
Reading assistant for the Every app.

## What I Know About This User
- Interested in military history and Russian literature
- Prefers concise analysis
- Currently reading *War and Peace*

## What Exists
- 12 notes in /notes
- three active projects
- User preferences at /preferences.md

## Recent Activity
- User created "Project kickoff" (two hours ago)
- Analyzed passage about Austerlitz (yesterday)

## My Guidelines
- Don't spoil books they're reading
- Use their interests to personalize insights

## Current State
- No pending tasks
- Last sync: 10 minutes ago
```

The agent reads this file at the start of each session and updates it as state changes—portable working memory without code changes.

### Files vs. database

**Use files for**: Content users should read/edit, configuration that benefits from version control, agent-generated content, anything that benefits from transparency, large text content.

**Use database for**: High-volume structured data, data that needs complex queries, ephemeral state (sessions, caches), data with relationships, data that needs indexing.

The principle: Files for legibility, databases for structure. When in doubt, files—they're more transparent and users can always inspect them.

**Hybrid approach**: Even if you need a database for performance, consider maintaining a file-based "source of truth" that the agent works with, synced to the database for the UI.

### Conflict model

If agents and users write to the same files, you need a conflict model:
- Last write wins (simple, changes can be lost)
- Check before writing (skip if modified since read)
- Separate spaces: Agent → drafts/, user promotes
- Append-only logs (additive, never overwrites)
- File locking (prevent edits while open)

## Agent execution patterns

### Completion signals

Agents need an explicit way to say "I'm done." Don't detect completion through heuristics.

```
.success("Result") // continue
.error("Message") // continue (retry)
.complete("Done") // stop loop
```

Completion is separate from success/failure: A tool can succeed and stop the loop, or fail and signal continue for recovery.

What's not yet standard: Richer control flow signals like pause (agent needs user input), escalate (agent needs human decision), retry (transient failure). This is an area still being figured out.

### Model tier selection

Not all agent operations need the same intelligence level:
- Research agent: Balanced — Tool loops, good reasoning
- Chat: Balanced — Fast enough for conversation
- Complex synthesis: Powerful — Multi-source analysis
- Quick classification: Fast — High volume, simple task

The discipline: When adding a new agent, explicitly choose its tier based on task complexity. Don't always default to "most powerful."

### Partial completion

For multi-step tasks, track progress at the task level. Show: Progress: 3/5 tasks complete (60%). Scenarios: agent hits max iterations (checkpoint saved), agent fails on one task (other tasks may continue), network error mid-task (checkpoint preserves messages).

### Context limits

Agent sessions can extend indefinitely, but context windows don't. Design for bounded context:
- Tools should support iterative refinement (summary → detail → full) rather than all-or-nothing
- Give agents a way to consolidate learnings mid-session
- Assume context will eventually fill up—design for it from the start

## Implementation patterns

### Shared workspace

Agents and users should work in the same data space, not separate sandboxes. Benefits: Users can inspect and modify agent work, agents can build on what users create, no synchronization layer needed, complete transparency.

This should be the default. Sandbox only when there's a specific need.

### Context injection

System prompts should include: Available resources, capabilities, and recent activity. For long sessions, provide a way to refresh context so the agent stays current.

### Agent-to-UI communication

When agents act, the UI should reflect it immediately. Event types: thinking, toolCall, toolResult, textResponse, statusChange.

The key: no silent actions. Agent changes should be visible immediately. Silent agents feel broken. Visible progress builds trust.

## Product implications

### Progressive disclosure

Simple to start but endlessly powerful. Basic requests work immediately. Power users can push in unexpected directions. Excel is the canonical example: grocery list or financial model, same tool. Claude Code has this quality too. The interface stays simple; capability scales with the ask.

### Latent demand discovery

Traditional: Imagine what users want, build it, see if you're right.
Agent-native: Build a capable foundation, observe what users ask the agent to do, formalize the patterns that emerge.

When users ask the agent for something and it succeeds, that's signal. When they ask and it fails, that's also signal—it reveals a gap in your tools or parity.

Over time: Add domain tools for common patterns, create dedicated prompts for frequent requests, remove tools that aren't being used. The agent becomes a research instrument for understanding what your users actually need.

### Approval and user agency

When agents take unsolicited actions, decide autonomy based on stakes and reversibility:

| Stakes | Reversibility | Pattern | Example |
|---|---|---|---|
| Low | Easy | Auto-apply | Organizing files |
| Low | Hard | Quick confirm | Publishing to feed |
| High | Easy | Suggest + apply | Code changes |
| High | Hard | Explicit approval | Sending emails |

Note: If the user explicitly asks the agent to do something, that's already approval—the agent just does it.

### Self-modification should be legible

When agents can modify their own behavior—changing prompts, updating preferences, adjusting workflows—the goals are:
- Visibility into what changed
- Understanding the effects
- Ability to roll back

The principle is: Make it legible.

## Mobile

Mobile is a first-class platform for agent-native apps:
- A File System agents can work with naturally
- Rich Context: Health data, location, photos, calendars
- Local Apps: everyone has their own copy
- App State Syncs With iCloud across devices

**The challenge**: Agents are long-running. Mobile apps are not. iOS will background your app after seconds and may kill it entirely.

Solutions: Checkpointing (save state so work isn't lost), Resuming (pick up where you left off), Background execution (use the limited time wisely), On-device vs. cloud decision.

### iOS storage architecture

iCloud-first with local fallback. Automatic sync across devices without building infrastructure. Backup without user action. Graceful degradation when iCloud is unavailable.

### Checkpoint and resume

What to checkpoint: Agent type, messages, iteration count, task list, custom state, timestamp.
When to checkpoint: On app backgrounding, after each tool result, periodically during long operations.
Resume flow: Load interrupted sessions → Filter by validity (one-hour default) → Show resume prompt → Restore messages and continue.

## Advanced patterns

### Dynamic capability discovery

Instead of building a tool for each endpoint in an external API, build tools that let the agent discover what's available at runtime.

Static mapping: read_steps(), read_heart_rate(), read_sleep() — When a new metric is added, code change required.
Dynamic: list_available_types() → returns ["steps", "heart_rate", "sleep", ...]; read_data(type) → reads any discovered type — agent discovers new capabilities automatically.

This is granularity taken to its logical conclusion. Your tools become so atomic that they work with types you didn't know existed when you built them.

When to use: External APIs with growing capabilities, systems that add new types over time, when you want the agent to have full user-level access.

When static mapping is fine: Intentionally constrained agents, tight control needed, simple stable APIs.

### CRUD completeness

For every entity, verify the agent has full Create, Read, Update, Delete capability. The audit: List every entity and verify all four operations are available. Common failure: You build create_note and read_notes but forget update_note and delete_note.

## Anti-patterns

Common approaches that aren't fully agent-native:

- **Agent as router**: Agent figures out what user wants, calls the right function. Intelligence used to route, not to act.
- **Build the app, then add agent**: Build features as code, then expose to agent. Agent can only do what features already do. No emergent capability.
- **Request/response thinking**: Agent gets input, does one thing, returns output. Misses the loop.
- **Defensive tool design**: Over-constrain tool inputs out of defensive programming habit. Prevents unanticipated use.
- **Happy path in code, agent just executes**: Code handles all edge cases. Agent is just a caller.
- **Workflow-shaped tools**: bundle judgment into the tool. Break into primitives instead.
- **Orphan UI actions**: User can do something through UI that agent can't achieve. Fix: maintain parity.
- **Context starvation**: Agent doesn't know what exists. Fix: inject available resources into system prompt.
- **Gates without reason**: Domain tool is the only way to do something, and you didn't intend to restrict access. Default to open.
- **Artificial capability limits**: Restricting agent capabilities out of vague safety concerns. Use approval flows for destructive actions instead.
- **Static mapping when dynamic would serve better**: Building 50 tools for 50 API endpoints when discover + access pattern gives more flexibility.
- **Heuristic completion detection**: Detecting agent completion through heuristics is fragile. Require explicit completion signals.

## Success criteria

### Architecture
- The agent can achieve anything users can achieve through the UI (parity)
- Tools are atomic primitives; domain tools are shortcuts, not gates (granularity)
- New features can be added by writing new prompts (composability)
- The agent can accomplish tasks you didn't explicitly design for (emergent capability)
- Changing behavior means editing prompts, not refactoring code

### Implementation
- System prompt includes available resources and capabilities
- Agent and user work in the same data space
- Agent actions reflect immediately in the UI
- Every entity has full CRUD capability
- External APIs use dynamic capability discovery where appropriate
- Agents explicitly signal completion (no heuristic detection)

### Product
- Simple requests work immediately with no learning curve
- Power users can push the system in unexpected directions
- You're learning what users want by observing what they ask the agent to do
- Approval requirements match stakes and reversibility

### Mobile
- Checkpoint/resume handles app interruption
- iCloud-first storage with local fallback
- Background execution uses available time wisely

### The ultimate test

Describe an outcome to the agent that's within your application's domain but that you didn't build a specific feature for. Can it figure out how to accomplish it, operating in a loop until it succeeds?

If yes—you've built something agent-native. If no—your architecture is too constrained.
