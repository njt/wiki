# Self-Hosted Sandboxed Agentic Software Factory

Jake Saunders' field report on building a personal, almost-fully self-hosted agentic development environment that takes a single prompt and autonomously walks the whole SDLC — research, code, tests, commit, CI, deploy — while *structurally containing* the LLM rather than trusting it. Running on a sacrificial eBay box with no external ingress, the stack (Coolify, Forgejo, Hermes, Firecrawl, Tailscale, Porkbun DNS-01) delivered a SvelteKit + Postgres app from paragraph to HTTPS deployment with one follow-up prompt, at an ongoing cost of a £20 Codex subscription.

---

## Key Quotes

> "So, the challenge: how can I create a fully remote agentic development environment where we structurally contain the LLM rather than just trusting it?"

The thesis in one sentence. The word *structurally* is doing the real work: containment here is a property of the topology (separate metal, no ingress, network-scoped credentials), not of instructions in the prompt. This is the same conclusion the [[Security and Sandboxing]] hub reaches from a different direction — enforce below the model, because a model processes trusted instructions and untrusted data through the same weights.

> "The failure mode is now 'rebuild the eBay box and rotate a handful of keys', rather than 'discover an LLM has enthusiastically reorganised my actual laptop'. That's better I think, but it isn't magic."

The honest cost/benefit statement that the whole exercise rests on. Saunders doesn't claim to have made the agent *safe* — he lists everything Hermes can still do (nuke the box, delete repos and databases, leak credentials, burn tokens, make arbitrary outbound requests, poke the rest of the network). He has bounded the *blast radius*, which is a weaker but achievable promise. This is the same "sacrificial machine" logic as [[Domenic Denicola's Agentic Coding Setup]], stated more explicitly.

> "The best bit is that Coolify does this on the fly. Our agent can create a service at any subdomain and it'll ✨magically✨ sort itself out."

The DNS-01 insight is the most genuinely novel engineering in the piece. By giving Coolify Porkbun API keys with write access to the domain, he gets valid HTTPS certificates for "ghost services" — reachable only on the tailnet, with *no public A/AAAA record* pointing at his IP. Deployment becomes something the agent can do unaided, without ever associating a subdomain with his home IP in public DNS.

> "At some point, though, enough approval gates turn your magical autonomous software factory back into a collection of forms you have to fill in. Finding the useful point between 'needs me every five minutes' and 'has the launch codes' is the next experiment."

The closing open question, and the sharpest one. This is the autonomy/safety dial that every factory builder eventually confronts — the same tension [[Cloud Software Factories]] names as "steering vs. the control room" and [[The Agent Access Model]] frames as "exceptional rather than per-action oversight."

## Key Themes

#concept #tool #pattern #self-hosting #sandboxing #software-factory #homelab

### The Sacrificial-Machine Pattern

Containment by blast-radius, not by permission. Saunders gives Hermes full control of *a* machine, then makes that machine cheap to lose — a second-hand i7 bought fresh from eBay with nothing on it. The first guardrail is "it's on its own metal"; the second is that there's no external ingress (no port 443 forwarded), cutting out both the attack surface and the internet background radiation. This is a hardware-first answer to the question [[How We Contain Claude]] and [[cco]] answer in software, and it's accessible to a solo homelabber in a way enterprise gVisor/VMs aren't.

### Networking as the Real Sandbox

Most of the article's novelty is network engineering, not agent engineering. Tailscale provides the private backplane and lets his phone reach services; Pi-hole's dnsmasq rules resolve `*.internal.jakeshomelab.me` to the new server; Coolify's Traefik reverse proxy serves them. The elegant piece is DNS-01 via Porkbun: Traefik creates a `_acme-challenge` TXT record, Let's Encrypt validates it, the record is deleted — so the agent gets `https://cool-new-app.internal.jakeshomelab.me` with no public A record. The one leak he concedes: the hostname still lands in public certificate-transparency logs.

### The Self-Hosted SDLC Stack

The tooling is well-known; gluing it together is the contribution. Forgejo (chosen over GitHub because a GitHub token would undermine isolation) stores code and runs CI with its own runners; Coolify's recipes deploy Postgres, Redis, Hermes, and Forgejo with one click, and do S3 backups in three; Hermes drives the agent with a Web UI, a Samba-shared filesystem, Telegram integration, and — notably — *self-built skills* (it read Coolify's docs and built its own Coolify skill). Self-hosted Firecrawl gives the agent SERP and scraping access.

### What It Actually Did

The demo prompt specified a MyFitnessPal-like calorie tracker (SvelteKit + Drizzle + Postgres + Tailwind, mobile-first, Docker Compose, deploy its own Postgres). The agent bootstrapped, wrote app and tests, built CI, iterated until green, containerised, and deployed — with no further prompting. One follow-up message fixed a CSRF bug, added regression tests, and redeployed. Saunders' verdict: "it went from a paragraph to tested, deployed software and handled all the boring bits in between."

## Critical Analysis

**The honest part is the best part.** Most "I built an autonomous software factory" posts stop at the demo video. Saunders spends the back half on the list of things Hermes can still do — including "burn through inference tokens like its end-of-year review depends on it" — and lands on a defensible, non-magical claim: he changed the failure mode, not eliminated it. That candor is what makes the piece useful as a threat model rather than a victory lap.

**The economics are quietly radical.** The only ongoing cost is a £20 Codex subscription for inference — no cloud infrastructure bill, no per-token metering. Compare this with [[What a User Story Actually Costs in a Dark Code Factory]], where a cloud factory prices a delivered story at a median $9.56 and the ledger's accuracy is a research problem in itself. The self-hosted factory inverts the economics: capital expenditure (a used box) instead of operating expenditure, and the "meter" is whatever the flat-fee subscription absorbs. The trade-off is that self-hosting moves the maintenance burden — 45+ Docker containers, DNS, certs, CI runners — onto the operator.

**It's a solo-developer factory, and that constrains what it proves.** The app is deliberately "as CRUD-y as it gets," and — crucially — it talks to nothing but its own Postgres. Saunders is explicit that "most useful software talks to other software, which means handing over API keys, and every key adds another little hole in the sandbox." The whole demonstration sidesteps the credential-scoping problem that [[The Agent Access Model]] and [[Corsair — Agent Integration Layer]] treat as the hard center of agent security. A calorie tracker is a proof of the loop, not of the containment.

**The self-building-skill detail is a live security question.** Hermes "read the docs, looked at the MCP and built one" — a self-improving agent writing its own capabilities, exactly the loop [[Personal Agents]] flags as unaudited and [[Malicious Agent Skills in the Wild]] measures as a real attack surface. Saunders notes that getting keys "in the right places is a massive pain in the arse," which is the mundane symptom of the deeper issue: a self-modifying agent with write access to a domain registrar and a code host is a supply chain of one.

**It's a waypoint, not a reference architecture.** Like [[Domenic Denicola's Agentic Coding Setup]], the value is a concrete, dated snapshot of what "works today" assembled from off-the-shelf parts — Coolify, Forgejo, Hermes, Tailscale — none of which he had to build. The next steps he names (own VLAN, scoped credentials, one-shot rebuild, approval gates for irreversible actions) are the parts that would make it durable, and they're all still to do.

---

*Sources: [[raw/building-an-almost-fully-self-hosted-sandboxed-agentic-software-factory]], [[summary/building-an-almost-fully-self-hosted-sandboxed-agentic-software-factory]]*
*Last updated: 2026-09-04*
