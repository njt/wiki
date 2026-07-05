---
url: https://www.anthropic.com/news/introducing-claude-tag
title: Introducing Claude Tag
author: Anthropic
date_fetched: 2026-06-24
date_published: 2026-06-23
---

Claude Tag is Anthropic's new approach for teams to collaborate with Claude, starting on Slack. Claude joins a Slack workspace as a team member, accesses selected channels, and connects to tools, data, and codebases. Users tag @Claude to delegate tasks.

## How It Works

When tagged with a request, Claude breaks the task down into stages and works through them using available tools, responding in a Slack thread.

### Four Key Advantages

1. **Multiplayer:** One Claude per channel interacts with everyone. Anyone can see its progress and pick up where others left off — "much more like interacting collaboratively with a teammate" rather than a single-chat experience.

2. **Learns over time:** Claude builds context by following channel activity, reducing re-explanation. Can pull from other channels and data sources if permitted. "It doesn't report from private channels."

3. **Takes initiative:** With "ambient" behavior enabled, Claude proactively surfaces relevant information and follows up on stalled threads or unresolved tasks.

4. **Works asynchronously:** Users set a task and move on. Claude can schedule tasks and run projects autonomously "over hours or days."

Users can also send Claude direct messages for private interactions with personal tools.

## Internal Usage at Anthropic

- 65% of the product team's code is produced by their internal Claude Tag version
- Usage has expanded beyond engineering into product metrics, data analysis, support tickets, and bug root cause analysis

## Pricing and Availability

- Beta for Claude Enterprise and Team customers
- Works with Opus 4.8
- Introductory launch credit for eligible organizations

## Setup and Administration

Four-step admin process:
1. Pair Claude Tag with the Slack workspace
2. Grant tool access
3. Set a monthly organization spend limit
4. Test in a private channel

Administrators control tool access per channel, creating separate "Claude identities" with scoped memories — a sales instance won't share context with an engineering one. Token spend limits per organization and per channel. Full activity log available.

Claude Tag replaces the existing "Claude in Slack" app; admins migrate via opt-in within 30 days.

## Key Quotes

- "65% of our product team's code is created by our internal version of Claude Tag"
- "we now spend much more of our time delegating tasks to many Claudes in parallel"
- "everything, including its memories, will stay scoped to the channels defined by the administrators"
