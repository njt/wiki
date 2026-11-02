# I Killed Unlimited ChatGPT

A practitioner explains how he turned ordinary unlimited ChatGPT chats into a pool of coding agents by wiring them to a home-built MCP gateway (Desktop Commander + computer use) over OpenAI's Secure MCP Tunnel, then orchestrating parallel chats against a queue of GitHub issues — and why OpenAI killing unlimited chats is sending him to Claude.

---

## What he built

The setup has three layers:

1. **Tools.** A unified MCP server running on his Windows laptop (with a twin on his VPS), integrating Desktop Commander (file access, terminal commands, test runs, process management) and a CUA integration (screenshots, mouse, clicking buttons, visual inspection of UIs).
2. **Transport.** OpenAI's Secure MCP Tunnel bridging the local gateway to ChatGPT, authenticated with an API key — no bespoke cloud infrastructure. He emphasises the whole stack cost him nothing beyond the subscription and hardware he already had.
3. **Visual feedback.** Custom MCP App plumbing so captured screenshots render *inside* the ChatGPT conversation rather than being model-visible-only, plus prettier tool-call results.

## The orchestration trick

ChatGPT chats can't spawn subchats or subagents. His workaround is the most interesting part of the piece: a strong reasoning model (Astra in Codex, or the Pro model) drafts a detailed implementation plan and decomposes it into GitHub issues — each self-contained enough ("enough context, requirements, and acceptance criteria") that an independent agent can pick it up. Then multiple parallel ChatGPT conversations on the cheaper "Sol 6 High" tier each grab an issue and work it via MCP against the shared filesystem, iterating on tests and errors. GitHub issues are the task queue; premium-model spend is rationed to architecture, hard decisions, and review.

He's honest about the seams: parallel agents sharing a filesystem need coordination, and some issues need a stronger model to intervene. For well-scoped tasks, though, it "worked surprisingly well." He also built a small dashboard for MCP connection control, server health, CPU/memory, and a live feed of tool calls — including failure tracking.

## The economics punchline

The whole edifice existed to arbitrage a pricing tier: unlimited chats on one side of OpenAI's product line, metered Codex usage on the other. When OpenAI merges the two, the arbitrage closes — and the author announces he's switching to Claude, which he already prefers on model quality anyway. OpenAI doesn't just lose the loophole; it loses the subscriber.

---

Key quotes:

> "I was essentially using ChatGPT as a pool of coding agents, with GitHub issues acting as the task queue."

Clean articulation of the pattern: the issues-as-queue move is what makes heterogeneous agents (strong planner, cheap workers) coherent. Same shape as kanban-driven agent management elsewhere in the wiki.

> "To be clear, I am not breaking any rules. I am simply making the best use of what OpenAI has made available to me."

A fair defence and an interesting product signal: users will compose any exposed primitives into whatever maximises their value, and the gap between "unlimited chats" and metered agent usage was a pricing bug, not a user misbehaviour. Vendors closing such gaps should expect churn, not gratitude.

> "Claude Opus in particular seems to be much smarter for my use cases and I also get more mileage out of a Claude sub vs a Codex sub."

Model preference plus per-token mileage — the churn isn't just about the loophole closing.

## Themes

#pattern #tool #concept

## Take

A first-person account that doubles as a pricing-strategy case study. The technically interesting part — cross-provider MCP tooling, screenshots-in-chat, plan-into-issues fan-out — is honestly a bit standard by late 2026; what makes the piece worth keeping is that it documents the *incentive surface* of subscription AI products. Users meter themselves more sensibly than vendors think (he saved premium models for the decisions that mattered), but they will arbitrage any inconsistency between two tiers ruthlessly, and closing the arbitrage without matching the value is how you convert a happy power user into a competitor's subscriber. It also underlines how MCP has quietly become the universal adapter that makes any chat surface a coding-agent surface — the harness is whatever tools the gateway exposes.

## Related pages

This strengthens [[Managing Agents via Kanban Boards]]: same issues-as-queue pattern, reached independently, with the extra nuance that a weak "worker" tier is fine if issues are written to be agent-independently executable. It nuances [[Designing an MCP Gateway]] with a homely counterpoint — a personal unified gateway on a laptop can be built from open-source parts and OpenAI's own tunnel, with a dashboard for tool-call visibility. And it complicates [[Parallel Coding Agents Guide]] with the filesystem-contention caveat: his parallel chats shared one machine, and coordination friction was the acknowledged limit.

---
*Sources: [[raw/i-killed-unlimited-chatgpt]], [[summary/i-killed-unlimited-chatgpt]]*
*Last updated: 2026-11-02*
