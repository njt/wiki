# All Your Agents Are Going Async

Knill diagnoses the architectural gap between where agents are going (async background operation) and the transport they're built on (single HTTP request-response). He splits the problem into durable state and durable transport, argues current solutions from Anthropic and Cloudflare only solve state, and pitches Ably's session-based approach as the missing half. The diagnosis is sharp; the prescription has a vendor bias but the framing is useful regardless.

---

## Key Quotes

> "the lifetime of an agent's work is decoupled from the lifetime of a single HTTP connection"

This is the thesis. Once you accept that agents will run crons, respond to webhooks, and continue while you sleep, HTTP's request-response model is the wrong abstraction. The agent doesn't stop existing when you close the tab.

> "a chatbot's worst enemy is page refresh"

Pithy and correct. Every serious agent user has lost work to this. It's a transport problem masquerading as a UX problem.

> "Right now, they go in a database and you have to poll for them with some session URL"

The current state of the art for async results. Polling a session URL is what you do when you don't have durable transport — it's a workaround, not a solution.

> "These solutions solve only one half"

The cleanest framing in the piece. Knill splits the problem into durable state (where agent context lives) and durable transport (how bytes move between agent and human). Anthropic and Cloudflare focus on the former. Nobody has both.

> "A 'session' with an AI should be a thing that humans and agents can connect to, and disconnect from at any time"

The vision statement. Sessions as persistent rooms that survive disconnection, device switches, and fan-out to multiple humans.

---

## Key Themes

**#concept Durable State vs Durable Transport** — The core distinction. Durable state is where agent context lives across restarts. Durable transport is how response bytes travel across disconnects, device switches, and server-initiated push. You need both. Current solutions only provide state.

**#pattern Agent Lifetime Decoupling** — The agent's work outlives any single HTTP connection. This isn't a bug to patch; it's a fundamental property of async agents. The transport layer needs to reflect this.

**#tool Ably Session Transport** — Ably's bet: layer session state and conversation history on top of their existing pub/sub messaging infrastructure. Bidirectional, durable, realtime. Disclosed vendor interest, but the architecture is sound.

**#tool Anthropic Routines & Remote Control** — Pulling session lifecycle management into the platform rather than leaving it to the transport layer. Consolidates agent state but still relies on HTTP polling for delivery.

**#tool Cloudflare Agents Platform** — Sessions API for durable conversation storage, Email for Agents for async notifications. Same pattern as Anthropic: durable state, not durable transport.

**#concept Four Async Failure Modes** — The four scenarios HTTP can't handle: agent outlives caller, agent wants to push unprompted, caller changes devices, multiple humans in one session. A useful checklist for evaluating agent infrastructure.

---

## Critical Analysis

**The diagnosis is better than the prescription.** Knill's problem-split (durable state vs. durable transport) is genuinely clarifying. Most discussion of agent infrastructure conflates the two. Separating them reveals that Anthropic and Cloudflare are both solving the same half — durable state — while nobody has cracked durable transport at platform scale. This is a real gap and Knill is right to call it out.

**But the vendor interest is material.** Knill works at Ably. The article is a pitch for Ably's approach, and it never seriously examines alternatives. WebSockets over a connection manager? gRPC streams with retry? BEAM/OTP's process model with supervision trees? The [[Process-Based Concurrency BEAM OTP]] page describes an actor model that's been solving durable transport for decades — and [[Loomkin]] is already applying it to agents. Knill doesn't mention any of this.

**OpenClaw's insight deserves more credit.** The piece treats OpenClaw as a stepping stone to Ably's vision, but OpenClaw's approach — piggybacking on existing async messaging infrastructure (WhatsApp, Telegram, Discord) — is arguably more pragmatic. These platforms already solved durable transport. The tradeoff is platform dependence, but for personal agents that's often acceptable. [[clawdBot]] and [[Rowboat]] follow the same pattern.

**The multiple-humans problem is underexplored.** Knill mentions it as one of four failure modes but doesn't return to it. A team of five working with one agent that pushes updates to all and accepts input from any — this is genuinely unsolved. [[Zero Alignment]] names the coordination failure mode; durable transport is the infrastructure prerequisite for fixing it.

**HTTP polling works better than the article admits.** For most current use cases — check on your agent from your phone, pick up a session you started at your desk — polling a session URL with a few seconds of latency is adequate. The article's case is strongest for push notifications (agent wants to tell you something) and multi-device fan-out, which polling handles poorly. But the urgency gap matters: if your agent finished a 20-minute task, does it matter whether you find out in 2 seconds or 30?

**The BEAM-shaped hole in the argument.** Erlang/OTP's actor model was designed for exactly this: processes that outlive connections, transparent distribution, supervision trees for failure recovery, and hot code reloading. The fact that agent frameworks keep reinventing these patterns from scratch — with HTTP polling and database-backed sessions — suggests the industry has forgotten lessons distributed systems learned in the 1990s. [[Distributed Systems]] and [[Process-Based Concurrency BEAM OTP]] are the relevant prior art that Knill's analysis would benefit from engaging with.

**The webhook standardization gap.** Knill's argument that agents need durable transport sits interestingly alongside [[Standard Webhooks]], which is trying to standardize the webhook primitive itself. If every webhook provider used the same signature format, retry semantics, and payload structure, building a durable transport layer on top would be dramatically simpler. The fragmentation problem isn't just at the transport level — it's at the webhook level too.

---

*Sources: [[summary/all-your-agents-are-going-async]]*
*Last updated: 2026-05-15*
