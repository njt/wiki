# Cloudflare Temporary Accounts for Agents

Cloudflare's `wrangler deploy --temporary` gives AI agents throwaway deployment targets with zero human sign-up — 60-minute ephemeral accounts that auto-provision Workers, issue API tokens, and deliver a claim URL for the human. It's the first major platform to treat "an agent needs to deploy" as a first-class use case rather than a human-credential-sharing afterthought.

---

## The Core Mechanism

Agents run `wrangler deploy --temporary` and Cloudflare provisions a temporary account behind the scenes. The key design choice: Wrangler doesn't require the agent to know about `--temporary` ahead of time. When an agent runs `wrangler deploy` without auth, Wrangler's CLI output tells the agent about the flag. The agent reads that output, adapts, and reruns correctly.

This is a beautifully minimal discoverability pattern. No API docs needed. No MCP server. Just a CLI that prints helpful instructions when it hits a known failure mode — exactly the kind of design [[10 Principles for Agent-Native CLIs]] calls Table Stakes.

> "When a human invites an agent to perform a task, deploying to a developer platform should be as natural and seamless as any other function the agent carries out."

The temporary account lasts 60 minutes. During that window the human can claim it via a sign-up/sign-in flow, preserving all created resources — Workers, databases, bindings, everything. Unclaimed accounts vanish.

## The Write → Deploy → Verify Loop, Unblocked

The post demonstrates the exact workflow agents need:

1. Human prompt: *"Make a hello world Cloudflare Worker in TypeScript and deploy it"*
2. Agent writes code, runs `wrangler deploy`, sees the `--temporary` hint
3. Agent reruns with `--temporary`, gets a token, deploys, curls the preview URL
4. Human prompt: *"Now change it to 'hello cloudflare' and redeploy"*
5. Agent edits source, redeploys using the same temporary account — no re-auth

This is the tight feedback loop [[Agent Coding Workflow]] describes as essential. Without temporary accounts, every deploy attempt hits a browser-OAuth wall that background agents can't cross.

> "Background AI sessions have no human in the loop, and are becoming the norm — any step that requires a web browser causes the AI to get stuck or choose to use some other tool, platform, or service."

## What's Actually New Here

Cloudflare isn't just adding a flag. They're solving three problems simultaneously:

### 1. Discovery without documentation

The `--temporary` flag is discoverable through CLI output, not external docs. An agent that knows how to use Wrangler (from training data) can discover the flag at runtime. This is smarter than it looks — it works with *any* coding agent, not just ones with Cloudflare-specific training.

### 2. Throwaway infrastructure as a primitive

Temporary accounts give agents "cheap, throwaway deployment targets" — infrastructure that's safe to create and safe to forget. This is the deployment equivalent of `mktemp -d`. Necessary for agents that generate, deploy, and verify in a loop.

### 3. Claim as the bridge back to humans

The claim URL is the elegant bit. The agent never sees a sign-up page. The human claims the account later, on their own time, in a browser. The agent's work isn't lost — it's preserved and handed off.

> "If unclaimed by that point, they are automatically deleted."

## Key Themes

#tool #pattern #concept #platform

## The Broader Bet

This post is a progress report on a larger thesis: that platform infrastructure must be redesigned for agent access, not just human access. Cloudflare cites two other ongoing projects:

- **Stripe partnership**: A co-designed protocol for agents to provision Cloudflare accounts on behalf of users — creating accounts, starting subscriptions, registering domains, getting API tokens — without copy-pasting tokens or entering credit card details.
- **WorkOS collaboration**: `auth.md`, letting agents provision accounts using existing OAuth standards rather than new proprietary auth flows.

The pattern is consistent: don't build custom agent auth. Make existing auth work for agents. The CLI-as-interface approach (Wrangler) and the OAuth-as-interface approach (WorkOS) are two sides of the same coin.

## Critical Analysis

**The good:** Cloudflare understood that the problem isn't "agents can't figure out OAuth." It's that OAuth was designed for browsers and humans-in-the-loop. The `--temporary` flag sidesteps the entire auth flow rather than trying to automate it. This is the right instinct — don't make agents better at pretending to be humans; make the platform speak the agent's language.

**The limitation:** 60 minutes is short. For a "hello world" example it's fine, but for anything involving iteration across multiple agent sessions, it's useless. The claim URL partially addresses this, but it still requires the human to notice the claim URL in the agent's output and act on it before the window closes. There's no webhook, no push notification — just a string in stdout.

**The real innovation isn't technical.** It's organizational. Cloudflare is treating "agents are users" as a product requirement, not a hackathon project. That's rarer than it should be in mid-2026. Most platforms are still in "here's an API key, good luck" mode. The Stripe and WorkOS partnerships suggest Cloudflare is building a systematic agent access layer, not just a feature flag.

**What's missing:** No mention of scoped permissions. A temporary account that can deploy Workers is one thing. A temporary account that can provision R2 buckets, modify DNS, or access existing zones is another. The post says "temporary accounts have some limitations" but doesn't enumerate them. The security model needs to be explicit — what can a temporary account *not* do?

**The comparison to [[InsForge]]:** InsForge gives agents a full backend (Postgres, auth, storage, functions) via MCP. Cloudflare's approach is different: the agent uses the same CLI a human would, but with a flag that changes the account model. InsForge is built *for* agents; Cloudflare is making an existing human platform *work for* agents. Both are valid, but Cloudflare's approach scales to more existing tools.

**Comparison to [[Computer Use is 45x More Expensive Than Structured APIs]]:** The browser-OAuth wall this post describes is exactly the cost the "computer use" article quantifies. Every time an agent hits a browser-required auth flow, you're paying 45x in tokens and 51x in time vs. a structured API call. The `--temporary` flag eliminates that entire cost category for Cloudflare's platform.

## Cross-References

- [[10 Principles for Agent-Native CLIs]] — The `--temporary` flag is a case study in Table Stakes: CLI output that teaches agents about capabilities
- [[Agent-Native Architectures (Every)]] — "Parity" and "composability" principles applied to deployment infrastructure
- [[Agent Coding Workflow]] — The write→deploy→verify loop that temporary accounts make possible
- [[Agentcookie]] — The parallel auth problem from the other direction: syncing human session state to agent machines
- [[Computer Use is 45x More Expensive Than Structured APIs]] — Quantifies the browser-OAuth friction this flag eliminates
- [[InsForge]] — BaaS built for agents (MCP-native); Cloudflare's alternative approach of making human CLIs agent-compatible
- [[Code Storage]] — Shares the thesis that agent-created infrastructure will dwarf human-created
- [[An Illustrated Guide to OAuth]] — The flow this flag elegantly bypasses
- [[Cloudflare Wallets]] — The companion product: Temporary Accounts solve deployment, Wallets solve payment and identity for the agentic Internet

---
*Sources: [[summary/cloudflare-temporary-accounts]]*
*Last updated: 2026-06-22*
