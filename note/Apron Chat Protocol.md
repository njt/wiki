# Apron Chat Protocol

Apron is a deliberately minimal protocol for self-hosted chat — JSON over any transport, a server writable in ~100 lines, and just enough structure for rooms, threads, bots, and reactions. Its interest for this wiki is less chat than the pattern: a tiny core spec with optional capabilities and graceful fallbacks, plus a two-frame bot API that makes agents first-class participants.

---

## What the protocol is

The entire core is a request/response JSON envelope over any transport. The homepage's handshake transcript is the spec in miniature:

> {"method":"server", "params": {"apron": 9, "auth": ["guest"]}}
> {"method":"auth", "id": "c1", "params": {"scheme":"guest", "name":"Ada"}}
> {"id": "c1", "result": {"you": {"user_id": "guest_1234", "name":"Ada"}}}
> {"method":"message", "id": "c2", "params": {"body": {"text":"Hello"}}}

Server advertises version and auth schemes; client authenticates; client sends a message; server echoes a `message_id` and the room broadcast carries `room_id` and `from`. "Small core specification and optional incremental capabilities with graceful fallbacks" is the stated design rule — capability negotiation without a second protocol.

## The ecosystem as the argument

Rather than a whitepaper, Apron argues through implementations:

- **Goapron-server-go** — reference server, every capability.
- **apron-server-cloudflare** — a Cloudflare Worker running the public demo on the free tier.
- **python-minimal** and **js-minimal** — one file each, no dependencies ("one room, guests, history").
- **apron-web** — the SvelteKit client behind the demo.
- **apron-pr-bot** — posts GitHub pull request activity to the demo server.

The bot example is the tell. Posting CI or PR activity is "two frames to post": one token auth, one message (`{"method":"message", ... "text":"Deployed v1.4.2"}`). Any script that can emit two JSON lines is a client.

## Key themes

#tool #pattern #concept

- **Small core, capabilities as increments.** The base protocol is a few methods; features layer on top with fallback. Compare MCP's much larger surface — Apron is the opposite bet on what a protocol can get away with.
- **Interoperability by poverty.** "Any client works with any server" holds because there is so little to disagree about.
- **Bots as the cheapest clients.** The PR bot shows the protocol doubling as a notification bus for agents and automations — a self-hosted, grep-friendly alternative to Slack webhooks.

## Opinionated take

Apron is a design-taste artifact, and a good one: it demonstrates how far "JSON over a socket with four verbs" gets before you need rooms-with-permissions, and the one-file server implementations are the proof rather than a promise. The honest criticism is that the hard problems of chat — identity across servers, history sync, moderation, delivery guarantees on flaky transports — live entirely in the "optional incremental capabilities" that the page doesn't show. Minimal protocols have a habit of staying minimal at the core while accreting a de-facto spec in the reference implementation. Still, for the wiki's concerns the bot framing matters most: this is what an agent-facing notification channel looks like when nobody is trying to sell you a platform — two frames, self-hosted, no vendor.

## Related pages

[[Chatto]] is the maximalist counterpoint: a self-hosted chat built on event-sourced NATS with projections and OCC — Apron shows what you get for ~100 lines, Chatto shows what the full engineering cost buys. [[Agent Host Protocol (AHP)]] attacks a similar "small protocol as substrate" problem from the agent-harness side, and the contrast in scope is instructive. The two-frame PR bot strengthens [[Continuous AI]]'s case that repo-adjacent automations need cheap, always-on destinations for their output — this is one such destination without a SaaS in the middle. And it nuances [[Personal Agents]]: the same minimalism that makes a personal chat server writable in an afternoon makes it writable *by* an agent, closing the loop on self-hosted personal tooling.

---
*Sources: [[raw/apron-chat]], [[summary/apron-chat]]*
*Last updated: 2026-10-24*
