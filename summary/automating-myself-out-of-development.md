---
url: https://www.thoughtfultechnologist.com/p/automating-myself-out-of-development
title: "Automating Myself Out of Development"
author: Nune Isabekyan
date_fetched: 2026-06-15
date_published: 2026-04-28
topics:
  - misc
---

# Automating Myself Out of Development

Author: Nune Isabekyan (ThoughtfulTechnologist on Substack)
Published: April 28, 2026

## Full Article

### Intro

The author begins by stating they are neither an "AI-fanatic" nor an "AI-doomer," referencing a previous article about their conflicted relationship with AI. They emphasize that creation requires making a mess first, and that tools like Claude Code need hands-on *usage* to discover possibilities, limitations, and one's own workflow.

### Phase 0 – Tabs of Terminals

The author describes starting with a simple synchronous Claude Code session on their local machine, brainstorming and implementing together, then reviewing PRs. They used Claude.md files, notes, skills, MCPs, and sub-agents. Eventually, they began opening multiple windows for parallel features, using worktrees, sometimes working on different projects simultaneously to avoid overlap. They mention the Superpowers plugin (brainstorm → spec → plan → implementation flow) and credit Lina Edwards's "Be the Gate" piece for the insight that each phase needs its own context. However, context-switch fatigue set in, as they could only genuinely stay attentive to 2–3 features.

They note the emergence of OpenClaw/Clawdbot/Moltbot, which they initially disliked due to security concerns, but eventually AWS made it a one-click Lightsail deployment. Conversations with Sergey Rysev pushed them to "take myself out of the equation," comparing it to management delegation skills. They set up an EC2 instance with SSM, using only native Claude Code modes to stay within legal bounds, and began working toward removing themselves from the loop.

### Phase 1 – Let's Get Out of Local Machine

Moving off the local machine was frustrating but necessary. To trust "allow all changes" mode, they wanted to reduce the blast radius by isolating projects to a single EC2. The migration revealed how repository contexts had leaked into each other and how their CLAUDE.md and memory files were "too inter-connected and messy." They felt slower and found Claude "being stupid" without prior conversation context. Automation initially made them slower, but it scoped potential damage to a single project rather than their entire developer machine. They struggled with sandbox mode and credential leakage concerns, noting that "Agent-env-as-a-service" environments are emerging.

The resulting flow: Brainstorm/Spec/Plan is collaborative (interactive Claude Code) → Review of Plan/Integration (interactive review with Claude Code) → Implementation/Test/Commit (automated Claude Code non-interactive) → Review of PRs/Revert if needed (human review).

The win: time not spent watching implementation of already-planned work, plus a cleaner machine state.

### Phase 2 – Let's Make It Work Stand-alone

The author tried keeping an interactive session open to the EC2 from their phone via remote terminal. It technically worked but failed as a human experience. Two reasons: they wanted Claude Code to run on a schedule (removing themselves from the loop), and they didn't want to work when away from the computer. They wanted **checkpoint-style** communication — Claude does a chunk, leaves a clear artifact and question, and they return the next morning.

This required: persistent state storage between runs, a way for the schedule to know what to pick up next, and clear stopping points with enough context for a quick response.

### Phase 3 – GitHub as the Board

After experiments with .md files and daemons, and conversations with Sergey about "giving his agents a planning board to work with," they settled on GitHub issue tracking (avoiding JIRA's heavy MCP and leveraging short-lived GitHub credentials). GitHub issues worked well because of labels (state machines), comments (daemon notes), a clean web UI for mobile reading, and a CLI (`gh`) for scripting.

The workflow was migrated: a backlog repo holds issues; labels represent phases; spec/plan artifacts live in `specs/issue-N/` directories. The skill `/feature-gh` knows how to brainstorm from an issue number, run spec review, plan creation, and plan review as isolated subagent passes, stop at hard gates awaiting a label flip, resume from `state.json`, and merge feature branches on command. Each phase has its own context window — the brainstorm subagent doesn't see implementation noise, the reviewer doesn't see brainstorming ramble.

### Phase 4 – Daemon First Version

Without ever using OpenClaw, the author independently arrived at the need for a `tick.sh` — a small bash script running on cron every 15 minutes on EC2. It: takes a lock, refreshes the gh token, pulls the backlog repo, resets stuck issues, finds the oldest `ready`-labeled issue, claims it by swapping labels and leaving a comment, spawns `claude -p` non-interactively with implementation instructions, and updates the issue label based on the result. The shell script is deliberately dumb — "The actual *intelligence* of the implementation is inside the Claude subprocess."

### Phase 5 – Actually Using It

The author's role became: brainstorm a feature with Claude during an active session, write the spec, accept the plan, get the issue to `ready`, then close the laptop. Mornings involved scanning for `needs-attention`, `branches-ready`, adding merge labels, and queuing the next night's batch. This was the first time it felt different — nighttime for coding, daytime for review and thinking. There were setbacks from token limits and time spent developing the process itself. Productivity didn't *feel* higher due to delays between thinking about a feature and seeing it work, but it helped clear the backlog. The bottleneck shifted from "I don't have time to write the code" to "I don't have time to brainstorm and review thoroughly enough."

### Phase 6 – Pre-context-gathering (enrichment)

The brainstorming step was consuming mornings. The author added an *enrichment* daemon pass: open a GitHub issue with one or two sentences, label it `needs-enrichment`, and the daemon expands it by reading relevant code, finding prior art, surfacing questions, and rewriting the issue body. It then stops at `enrichment:needs-review` for human approval. This moved context-gathering from human time to background time, so morning brainstorming started from "here's the code area, here's prior art, here are the open questions."

### Phase 7 – What If I Let It Auto-brainstorm Too?

This was the most cautious step. The auto-brainstorm pass only runs when explicitly opted in via label. It produces three artifacts: a frozen baseline spec, an editable working spec, and a "brainstorm log" — a Q&A receipt for each section with confidence levels and sources. The brainstorm log earned trust because the author can scan it in minutes and see exactly where the model was guessing. The human accepts the spec or opens an interactive editing session. When approved, a third daemon pass distills edits, cross-references changes with confidence levels, and writes corrections as principles for future runs while drafting a plan and review.

Three human gates remain in the auto path: confirm the enriched issue body, accept or edit the simulated spec, and accept or edit the auto-drafted plan. Implementation and merge remain daemon-driven; the merge step is always opt-in per issue.

### Phase 8 – The Current State

The full happy path includes five human touch points:
1. Write a brief issue (needs-enrichment)
2. Review the enriched issue body
3. Review the spec (accept or edit)
4. Review the plan (accept or edit)
5. (After daemon implementation) Review the diff and add merge label

On failure (conflict, broken build, exhausted tokens, unrecoverable state), the daemon labels the issue `needs-attention` and leaves a comment with breadcrumbs. No push notifications — just an extra label in the morning triage.

### What's Next

The author is unsure if this approach takes longer and costs more than direct implementation. They don't fully buy the promise of endless productivity, comparing it to being "bad at delegating to humans."

Clearly foreseeable next steps:

- **QA as the next bottleneck:** Tests are uneven and tend to over-mock. Future work around test design with reviewer agents checking what's *not* tested, integration coverage, and regression checks.
- **More reviewer/cleanup agents:** A tech-debt-suggester looking at the whole repo over time, an architecture reviewer, and a thorough security review pass.
- **Better categorization of features:** Small features shouldn't need 100 different reviews; the orchestrator should bucket them and adjust flow accordingly, with a separate bugfix flow.
- **More Meta:** An agent that suggests features and orders them by isolation level — but the author warns this enters the "broken telephone" territory Adrian Hornsby wrote about.

### A Note Before I Sound Too Convinced

The author firmly believes one must not delegate *thinking* to AI. Even this level of delegation is dangerous and produces technical debt. Some tests from the pipeline are bad; some architectural choices must be manually fixed. The pipeline doesn't remove the need for static analysis, code review, architecture review, rework, or security audits — it makes those *more* important because mediocre code can land faster. Throughput is higher but average quality is not.

"I keep going because I want to see how far this goes — what's still possible to delegate without crossing the line" into having the model do the thinking while the human merely signs off. The author doesn't know where that line is and expects to find it by overshooting and walking back.

The article concludes: "If anyone tells you with 100% confidence how AI must be used in your development process or organisation, run. They haven't tried it themselves."

### References

The author credits Mae Capozzi (and the Honeycomb team) for writing about useful skills and orchestrators; Sean Miller for questioning hands-on AI-assisted coding approaches; and Eric Lubow for writing about organizational dynamics effects. The article also references a Part 2.
