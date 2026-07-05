---
url: https://doneyli.substack.com/p/i-built-an-ai-chief-of-staff-that
date_fetched: 2026-07-05
backfilled: true
---

# I Built an AI Chief of Staff That Runs My Life While I Sleep

### This is what it looks like having an autonomous agent

I was drowning.

Four email accounts. Four calendars. Two kids in activities. A wife running her own practice. Monthly community events I organize. A Principal AI Architect role at ClickHouse with customer calls across time zones and travel weeks that cascade into scheduling conflicts I don’t see coming.

Things were slipping through the cracks. A missed registration for my daughter’s gymnastics. An email from a VIP contact that sat unanswered for a week because it got buried under 47 vendor pitches and automated receipts. A calendar conflict I didn’t catch until I showed up at two things simultaneously. So much noise that I couldn’t separate the signal in my own life.

Late last year, I started building my own Chief of Staff. Not a chatbot. Not an assistant that waits for commands. I wanted someone (something?) who could keep track of my life, learn about it, and offload everything that didn’t need my judgment.

I wrote a persona document that starts like this:


You are Don’s Executive Assistant (EA), a trusted gatekeeper and force multiplier. You protect his attention, surface what matters, and handle the rest.

Core principles: Protect Don’s attention ruthlessly. Never leave ambiguity. Surface what matters, summarize the rest. Family and VIPs always get through.

On December 29, 2025, I made my first commit. OpenClaw (formerly Clawdbot, now 313K GitHub stars) was launching around the same time. I followed it closely while building mine. The project is impressive. But I wanted full control over security and to understand every primitive from scratch. When your agent sends an email it shouldn’t have at 2 AM, “I don’t know how this layer works” is not an acceptable answer.

313 commits later: 43,000 lines of Python. 25 database tables. 133 test files. Seven background jobs running 24/7. Last month I compared the two. Capability overlap: over 80%. I keep an OpenClaw instance running in isolation for research. No ego. But my production system is the one where I know every line of code.

**Why build your own when OpenClaw exists?** Because for the first time, you can build **personal software, 100% customized to your life.** When I needed location-aware scheduling, I told the system to learn it. When my wife needed her own inbox managed in her own voice, I built it that weekend. When my daughter’s activities needed tracking, I added family calendar intelligence. The Chief of Staff itself helps me build its next capability.

This is not a weekend project. This is production infrastructure for my life.

And it’s not the only agent I run. The Chief of Staff is one node in a fleet of autonomous agents running across my infrastructure:

- **AI Chief of Staff (7 agents):**Email triage, calendar, briefings, memory reflection, Signal bot, watchdog
- **Local Infrastructure (9 agents):**Docker orchestration, backups, disk/SSH/network monitoring, always-on watchdog
- **Content & Research (10 agents):**Signal scanning, content planning, analytics, engagement monitoring, lead detection

All launchd-native. All running on a repurposed 2022 MacBook Pro that I turned into a dedicated “Agent Server.” You don’t need new hardware for this. The tech stack:

- **Hardware:**Old MacBook Pro (M1, 2022), lid closed, running 24/7 with- `caffeinate`
- **Scheduler:**macOS launchd (native, no cron, survives reboots)
- **Runtime:**Docker for ClickHouse, Langfuse, Postgres. Python + Poetry for agent code.
- **Networking:**Tailscale for private encrypted access from anywhere. SSH hardened. No ports exposed to the public internet.
- **Notifications:**ntfy (self-hosted push notifications) + Signal (encrypted messaging)
- **Observability:**Langfuse (LLM traces), Gatus (uptime monitoring), ClickHouse (analytics)
- **LLM cost:**Claude Max ($100/month). 26 agents, unlimited overnight runs, zero marginal cost per job. Try hiring a human EA for that.

Today I’m going deep on the Chief of Staff, the one that manages my life. But the patterns (two-tier processing, graduated trust, memory with decay) apply to every agent in the fleet.


Not a developer?Keep reading anyway. The architecture and trust model apply to any AI agent, including ones you can set up without writing code. I’ll give you a 10-minute quickstart with Claude Cowork near the end of this post.

Developer who wants the implementation details?I’ll walk through the two-tier processing architecture, the trust graduation code, and the three-layer memory system with real Python snippets below.

**What I Tried First (And Why It Burned Money)**

The first version was a monolithic script. One big Python file. It checked email, called Claude for every single message, and dumped the results into a text file.

Four problems killed it:

**Cost.** Receipts, password resets, the same spam I get every Tuesday. All of it hitting the API. That’s like paying a senior consultant to sort your junk mail.

**Latency.** Everything ran in a single daily batch. An urgent email from my VP at 9 AM waited until the 5 PM run, making the agent useless for the one category that actually mattered.

**No learning.** The agent made the same classification mistakes repeatedly because it had no memory. It couldn’t learn sender-level rules or flag patterns. Every run started from zero context.

**No trust boundaries.** The script had full permissions from day one. Read, draft, send. One bad prompt injection and it could send messages to anyone in my contacts. I covered the security implications in Build Log #7, but the architectural problem was deeper: no concept of earned trust. Binary. On or off.

I scrapped the monolith and started over with a question: what would it look like to build an agent that earns autonomy the way a new hire does?

**The Architecture That Never Sleeps**

The system has two operating modes, and this distinction matters:

**Autonomous mode:** handles the predictable. Scheduled jobs run around the clock: urgent email scanning every 30 minutes (no LLM, pure rules), full email triage at 5 PM (LLM classification + draft generation), daily briefing at 5:30 PM, and nightly memory reflection. You don’t touch it. It just works.

**Interactive mode:** is when you need something now. Text the Signal bot from your phone, in natural language, over encrypted messaging. It spins up a Claude Code CLI session with 75 MCP tools covering your full Google Workspace. Each tenant gets their own isolated session.

The core insight was separating urgency detection from deep analysis. Not every email needs an LLM. Most emails need a rule.

**Tier 1** runs every 30 minutes. No LLM. It scans against my contact list, checks for urgent keywords, and applies Gmail label matching. Alerts on my phone within seconds. Cost: zero.

**Tier 2** runs once daily at 5 PM. The LLM pulls the ~50 emails that accumulated, classifies each one (reply_urgent, reply_normal, read, action_required, archive), and generates context-aware drafts using memory from previous interactions.

The two-tier split cut my LLM costs by roughly 80%. Most emails need pattern matching, not intelligence.

**If you’re building agents in your organization, this is the pattern:** deterministic fast path for the 80% that follow predictable rules, LLM calls only for the 20% that require actual judgment.

One more choice worth noting: two Google transport layers (SDK for email and calendar, GWS CLI for Drive, Docs, Sheets, Tasks, and Contacts), both switchable via environment variable. The MCP server exposes 75 tools so Claude sees one unified interface regardless of transport.

**The Graduated Autonomy Model**

When enterprises deploy AI agents, I keep hearing the same question: “How much autonomy should we give it?” The answer is never binary. It’s graduated.

The Graduated Autonomy Model has three levels of trust, and an agent must earn its way up through measurable performance:

**Level 1: Approval Required**

Every draft gets queued. The human reviews and sends manually. The agent is in training mode.

**Graduation criteria:** 20+ drafts approved, less than 20% edit rate (meaning the human accepts most drafts without changes), more than 80% send rate, minimum 7 days of tracking.

**Level 2: Supervised**

The agent can auto-send, but only when ALL of these conditions are met simultaneously: edit distance below 10%, confidence score above 0.9, fewer than 5 auto-sends per hour, and the contact is NOT in a protected category.

**Graduation criteria:** 50+ successful auto-sends, 14 consecutive days with zero flagged issues, zero errors.

**Level 3: Autonomous**

Full auto-send for qualifying contact categories. But VIP and family contacts are hardcoded into a `NEVER_AUTO_SEND` list that no amount of good performance will override. This is a safety gate, not a trust level.

```
NEVER_AUTO_SEND = ["vip", "family"]
@dataclass
class GraduationThresholds:
    # Level 1 → Level 2
    approval_to_supervised_min_drafts: int = 20
    approval_to_supervised_max_edit_rate: float = 0.20
    approval_to_supervised_min_send_rate: float = 0.80
    # Level 2 → Level 3
    supervised_to_autonomous_min_auto_sent: int = 50
    supervised_to_autonomous_days_without_issues: int = 14
    # Time gates
    min_days_before_graduation: int = 7
    decay_window_days: int = 90
```
The decay window is critical. Only the last 90 days count. If my agent starts producing worse drafts, old successes don’t prop up its trust score. Trust is a rolling window, not a lifetime achievement.

And trust can be revoked. The system checks demotions BEFORE promotions. If edit rates spike or send rates drop, trust gets revoked immediately back to full human approval. Safety first, always.

**For enterprise leaders: this is the model.** Don’t ask “should our agent be autonomous?” Ask “what evidence would make us comfortable increasing autonomy, and what triggers would make us revoke it?” The Graduated Autonomy Model gives you a concrete answer to both questions.

**How It Remembers Everything (Without Blowing Your Token Budget)**

Here’s where most agent projects get stuck. An agent without memory is a stateless function. My agent has a three-layer memory system that gives it persistent context without blowing up the token budget.

**Layer 1: Observations.** Auto-captured at every integration point: draft corrections, classification overrides, contact-level rules you set explicitly. Zero LLM cost. Just structured logging.

**Layer 2: Memories.** A daily LLM reflection job synthesizes observations into durable memories. Weekly consolidation merges redundant ones. This is where raw data becomes institutional knowledge.

**Layer 3: Retrieval.** FTS5 full-text search with BM25 ranking. When the agent processes an email, it queries memory across four tiers, with a hard cap of ~550 tokens:

```
# Four-tier context injection (~550 tokens max)
context = []
context += get_core_memories(limit=3)        # "Don prefers concise replies"
context += get_sender_memories(sender, limit=3)  # "This sender is in the VIP list"
context += get_topic_memories(subject, limit=2)  # "Previous thread about Q2 planning"
context += get_situational_memories(limit=2)     # "Don is traveling this week"
```
The 550-token cap isn’t arbitrary. Below 400, the agent loses context and makes worse classifications. Above 700, quality plateaus while cost scales linearly. 550 is the sweet spot.

**The decay system prevents context pollution.** Every memory has a relevance score that decays over time:

- Recently accessed: 0.02 per cycle (slow decay) 
- Normal memories: 0.05 per cycle 
- Long-forgotten: 0.08 per cycle (aggressive cleanup) 
- Pinned memories: immune to decay 

Without this, a long-running agent accumulates stale context that pollutes its judgment.

**For teams building agents with persistent state:** the three-layer model separates zero-cost capture (Layer 1) from daily LLM synthesis (Layer 2) from search-based retrieval (Layer 3). Memory that scales without scaling your API bill.

**Beyond Email: The Part Nobody Talks About**

Here’s what surprised me. Email was the hardest problem to solve, but it’s maybe 40% of what the system does now. The real value turned out to be everything else.

**Calendar intelligence.** The system orchestrates all four calendars, detects conflicts, handles auto-RSVP based on sender tier and current mode (focus, travel, vacation), and pulls attendee history before meetings.

**Location awareness.** The agent calls the Google Maps API to calculate real commute times between meetings. A 2 PM downtown Montreal and a 3:30 PM across the river in Longueuil is either feasible or it isn’t, and the calendar gap alone won’t tell you.

**Signal bot in action.** I’ve been on a plane, texted “check my inbox,” and gotten a summary of what matters before landing. The five-tier dispatch (slash commands → intent router → full Claude Code session) means simple requests resolve instantly and complex ones get the full 75-tool MCP suite.

**Daily briefing.** The 5:30 PM digest (email + Signal) is structured: Action Required, Scheduled Events, FYI, Handled, Meeting Prep. Fridays include weekly insights. Both tenants get their own, independently.

**Running errands for me.** If I need to check something that requires a browser, the Chief of Staff has access to a secure Chrome session on my server. I can send it off to take action on my behalf, no banking obviously! But I’ve gotten it to submit forms to sign up for activities, pre-register in the website of my kids’ sports, etc. 

**The Multi-Tenant Test: When My Wife Became My Hardest Customer**

When I showed my wife the agent, her first question was “Can it do mine too?” Multi-tenant wasn’t a feature request. It was a marriage survival strategy.

This is also one of the biggest reasons I stuck with building my own. I needed true tenant isolation from day one: separate database, contacts, memories, persona, LLM client, OAuth credentials. Same codebase, zero data crossover.

And I got it wrong. Multiple times.

First bug: her briefing was delivered to my email address. A shared environment variable bleeding through the config merge. In production, that’s a data leak.

Second: the Signal bot was initialized with my MCP tools and my database. Her free-text questions would have queried my email, not hers.

Third: every draft signed off “Best, Don.” The persona file existed per tenant, but nobody was loading it into the LLM calls.

Each required a separate fix: per-tenant MCP configs, persona injection into every LLM pipeline, briefing recipient validation, and a `reflect-all` command that runs memory reflection for every tenant, not just the first.

Here’s what made it worth the pain: having a real second user made the system dramatically better. My wife doesn’t care about architecture. She cares about whether the draft sounds like her, whether the briefing shows her meetings, and whether the agent remembers that her colleague Zoe always needs a reply the same day. That kind of feedback is impossible to simulate.

**For teams building multi-agent or multi-user systems:** tenant isolation touches auth, config, memory, persona, notifications, and every scheduled job. It is not a feature you bolt on later.

**The Numbers**

After running for months in production:

- **Emails processed:**~50/day per tenant (100+ across both inboxes)
- **Tier 1 alerts:**Averages 3-5 urgent catches per day, delivered in under 10 seconds
- **Draft accuracy:**Above 80% send rate without edits for graduated categories
- **LLM cost:**Under $3/day for both tenants combined (dual-model strategy: Haiku for classification, Sonnet for drafting)
- **Uptime:**7 launchd jobs with watchdog auto-restart for the Signal bot
- **Memory:**~2,000 active observations, ~400 synthesized memories, decay keeps it clean
- **Test coverage:**133 test files, 396 passing tests (including the 52 security tests from Build Log #7)

**Build This Yourself (Or Audit What You Already Have)**

Whether you’re building from scratch, running OpenClaw, or evaluating any personal AI agent, here are the three patterns that matter most:

**1. Separate urgency from intelligence**

Don’t send every input through an LLM. Build a deterministic fast path for the predictable 80%, save the model calls for the 20% that need judgment. For email, calendar, or notifications, that split cuts costs by 60-80% and surfaces urgent items in seconds instead of hours.

**If you’re running OpenClaw:** Check whether your message routing sends everything to the LLM — if your grocery delivery and your VP’s urgent message hit the same pipeline, you’re overpaying.

**2. Graduate trust, don’t grant it**

Never give an agent full autonomy on day one. Start at Approval Required. Track edit rates, send rates, and error rates. Graduate to Supervised only when the numbers justify it. Hardcode your VIP and family contacts into a `NEVER_AUTO_SEND` list regardless of trust level.

**If you’re running OpenClaw:** If it can auto-send to anyone the moment you enable it, that’s the gap between a demo and a system you trust with real relationships.

**3. Build memory that decays**

A long-running agent without decay accumulates stale context. Implement the three-layer model: capture at zero cost, synthesize daily with an LLM, retrieve via search with a hard token cap. Decay rates ensure old memories fade unless accessed regularly.

**If you’re running OpenClaw:** Check whether it accumulates context indefinitely or has decay built in — unbounded memory eventually pollutes its judgment.

**Not building agents yet? Start here in 10 minutes.**

You don’t need 43,000 lines of Python to get value from this pattern. Claude Cowork can automate a simple email + calendar workflow right now, no code required.

Here’s a starting point:

- Open Claude Cowork and connect your Google account (Gmail + Calendar) 
- Ask: “Check my inbox for any emails from the last 24 hours that look like meeting requests or calendar invites. For each one, check my calendar for conflicts and draft a short reply.” 
- Review what it finds. Approve, edit, or reject each action. 
- You can put this on a schedule as a recurring task :) 

That’s it. The gap between this and a full Chief of Staff is automation, memory, and trust graduation. But the pattern is identical: scan, classify, draft, review.

I’m writing a full guide on going from Chat to Action with Claude Cowork. That’s a future Build Log.

**What’s Next: The System That Learns What You Need Before You Ask**

Everything I’ve described so far is reactive. The agent waits for inputs, then processes them. I’m building the next evolution right now: **proactive intelligence.**

Four phases, all shipped this week:

**Phase 1: Passive signal expansion.** Seven new observation types wired into the memory layer, all at zero LLM cost. Suddenly the system has data it never had before.

**Phase 2: User model.** A persistent behavioral model computed from accumulated observations across five dimensions: work patterns, decision patterns, relationship graph, bandwidth signal, and priority signals. The model gets injected into reflection prompts so the memory system can reason about behavioral changes.

**Phase 3: Anticipation engine.** This is where it gets interesting. The system now predicts needs instead of waiting for instructions:

- “You haven’t replied to [VIP contact] in 6 days. That’s 2x your normal pattern.” 
- “You made a commitment in Tuesday’s email to deliver a proposal by Friday. Your calendar shows zero time blocks for it.” 
- “Meeting load is 40% above your weekly average. Consider switching to focus mode.” 
- For senders where I accept drafts unedited more than 90% of the time, it pre-drafts responses before I even ask. 

**Phase 4: Daily learning digest.** One Signal message per day: “Here’s what I learned today.” New memories created, patterns detected, confidence milestones hit, uncertainties flagged. I can reply `wrong: 3` to dismiss a bad insight, or just text a correction. The feedback loop closes.

The cost of all four phases combined: roughly $0.008/day in incremental LLM spend. Less than a penny.

Twelve PRs. One epic. Zero increase in my daily costs worth measuring.

**For enterprise teams: this is the roadmap.** Most organizations stop at “our agent can answer questions.” The competitive advantage isn’t the agent that responds. It’s the agent that notices you forgot to respond.

**Coming soon in future Build Logs:**

**Kids activities management.** School pickups, hockey practice, birthday parties. Connecting existing calendar, location, and availability capabilities. AND signing up automatically to upcoming events! 

**Security deep-dive.** The Chief of Staff’s specific prompt injection defense, PII scrubbing, and anomaly detection architecture. A future paid Build Log.

The most autonomous system I’ve built has more guardrails than anything I’ve shipped to production. Turns out that’s not a contradiction, it’s the whole design.

Know a builder who would actually run this? Forward it to them.


Starting next week, Signal over Noise moves to Thursdays.Same depth. Same practitioner focus. Better day for your inbox.

Building something parallel to this -- mine started as a learning experience and it still is, but it's evolving into something genuinely useful. A month in, the successes and failures have been illuminating in equal measure. Different mission than yours: mine is career leverage and independence-building rather than life logistics.

Glad to see someone else who didn't just deploy OpenClaw. I'm sure it's great -- the guy got a new job out of it. But a system I didn't build isn't really my system. I want my chief of staff aligned to me, and when something breaks I want to know exactly how to fix it. There's accountability in that too: if the system does something wrong, that's fully on me. My API bill this month shows how much tuition I've paid.

One thing I've learned the hard way: context management is the real job. Not prompt engineering, not model selection -- managing what the system knows and forgets across sessions without losing the thread. Every meaningful insight has to get promoted to a persistent vault or it evaporates. Your memory reflection agent must be doing something similar. My favorite day was when I let the context eat all the memory on the machine.

Which leads to my question: what's the most surprising thing the system learned about you that you didn't expect to surface? Not a capability that worked -- a pattern about yourself the agent caught that you hadn't noticed.

Really liked the two-tier split here: cheap deterministic urgency detection first, LLM only when judgment is actually needed.

I'm building Watchline from a similar place, but at the event layer: let the agent register things like VIP email, calendar conflict, external-attendee meeting, then wake it only when the source event matches instead of running another polling loop.

The part I'm still thinking through is the boundary you described well: when should a match interrupt you immediately vs just land in the daily digest? Are you mostly deciding that with sender tier + keywords today, or has it gotten more nuanced over time?
