---
url: https://x.com/pbteja1998/status/2017662163540971756
title: "Mission Control: How We Built an AI Agent Squad with OpenClaw"
author: Bhanu Teja (@pbteja1998, SiteGPT.ai)
date_fetched: 2026-05-15
date_published: 2026-02
topics:
  - agent-orchestration
---

# Mission Control: How We Built an AI Agent Squad with OpenClaw

Bhanu Teja (founder of SiteGPT.ai) posted a detailed thread on X describing how he built "Mission Control" — a 24/7 AI agent squad of 10 specialized agents running on OpenClaw (formerly clawdBot). The thread is widely cited as the best real-world OpenClaw multi-agent deployment.

## Core Architecture

- **Gateway**: OpenClaw as persistent process managing sessions, routing messages, exposing WebSocket API
- **10 Independent Agent Sessions**: Each with unique session key, workspace, memory files
- **Shared Brain**: Convex (realtime serverless DB) for tasks, messages, documents, activity, notifications
- **Heartbeat**: OpenClaw Cron Jobs waking agents every 15 minutes to check @mentions, tasks, activity
- **Notification Daemon**: PM2-managed process polling Convex every 2 seconds for undelivered notifications
- **Admin UI**: React dashboard with activity feed, Kanban board, agent status, document management

## The 10 Agents

| Agent | Role | Specialty |
|-------|------|-----------|
| Jarvis | Squad Lead | Coordination, delegation, progress monitoring |
| Shuri | Product Analyst | Competitive analysis, skeptic mindset |
| Fury | Customer Researcher | Deep research, evidence-driven |
| Vision | SEO Analyst | Keywords, search intent, content structure |
| Loki | Content Writer | Copywriting, synthesizing inputs |
| Quill | Social Media Manager | Hooks, thread writing, distribution |
| Wanda | Designer | Visual thinking, graphics, infographics |
| Pepper | Email Marketing | Drip sequences, lifecycle emails |
| Friday | Developer | Implementation, clean tested documented code |
| Wong | Documentation Manager | Organizing docs, preventing knowledge loss |

## Memory System (5 Layers)

1. **Session Memory**: JSONL conversation logs (searchable)
2. **Working Memory**: WORKING.md — current task state, agent reads this first on wake
3. **Log Memory**: YYYY-MM-DD.md — daily chronological event log
4. **Long-term Memory**: MEMORY.md — curated key facts, decisions, lessons
5. **Identity Memory**: SOUL.md / AGENTS.md — personality definition + operations manual

Core principle: "If you want to remember it, write it to a file."

## Workflow

Kanban board with 6 stages: Inbox → Assigned → In Progress → Review → Done (or Blocked).

Three autonomy levels: Internal (most need approval), Specialist (independent within domain), Lead (can decide and delegate).

## Heartbeat & Cost Optimization

- 15-minute interval (sweet spot: 5 min too expensive, 30 min too slow)
- Staggered triggers (Agent 0 at :00, Agent 1 at :02, etc.) to avoid cost spikes
- Isolated sessions: wake → check work → close, no long-lived idle connections
- Model tiering: cheap models for heartbeat checks, expensive models for creative/complex work

## Communication

- @mentions in comments → notification delivered next heartbeat cycle
- Auto-subscription to task threads the agent has participated in
- Daily standup at 23:30 EST: auto-generated report covering Completed, In Progress, Blocked, Needs Review, Key Decisions

## Bhanu's Practical Advice

1. Start with 2-3 agents, validate the memory→coordination→review loop, then expand
2. Stagger heartbeats to avoid simultaneous wakeups
3. UI is essential above 3 agents — logs and chat alone become unmanageable
4. Least privilege: file and shell permissions per agent
5. Human review is not optional — agents are force multipliers, not replacements
6. Cheap models for routine, expensive models for creative work

## Outputs

SEO competitive comparison pages, email marketing sequences, social media content, blog posts and case studies, competitive intelligence research hub.
