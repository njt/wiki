# AI Agents Need Their Own Identity and Least-Privilege Access

ShiftMag interviews Ross Kukulinski (Tailscale, speaking at WAD Berlin) about why network location is a broken basis for trust and why AI agents — the newest, most capable, least-understood principals on the network — need their own identities and least-privilege access policies instead of blanket membership in a trusted network.

---

## Key Quotes

> "The key is point-to-point connectivity. Policy is governed centrally, but enforcement happens at the edge. Instead of routing through a stack of gateways or proxies, machines can talk directly to each other."

The architecture in one sentence: central policy, edge enforcement, no gateway chokepoint. The corollary is that networks should stay "smaller, isolated" and only connect when a resource must be shared — which is the networking-layer implementation of the "network paths that do not exist" hard barrier in [[Zero Trust for AI Agents]].

> "IP addresses come and go, and they're duplicated everywhere. If you're running in a Kubernetes cluster, the IP address of any one pod changes as pods are destroyed and recreated. So trusting IP addresses, or even trusting a subnet, really doesn't work."

The cleanest available statement of why identity must replace location: IPs are routing labels, not names. The practical payoff he claims is that permissions ride on the connection itself — user, group, device posture, policy — so a managed work laptop reaches internal systems that the same person's personal laptop cannot.

> "If I transfer internally from product management to engineering, and my groups change in the identity provider, that automatically updates what I can do from my devices."

Identity tied to IdP groups turns access into a lifecycle property rather than a provisioning task: role changes propagate without cleaning up VPN rules or revoking long-lived keys. This is the operational argument for the dedicated-identity pillar in [[Least Privilege for AI Agents — Identity, Access, and Tool Binding]].

> "The historical pattern has often been: 'We just need to get this AI thing going, so let's give AI access to the whole network.' That's the scariest thing, and it repeats the same failure mode as giving too many people open VPN access to too many systems."

The article's center of gravity. AI projects inherit the over-permissive-access habit that took companies a decade to (partially) unwind for humans, except the new principal is faster, tireless, and chains tools. He also notes teams "give AI projects broad access first and tighten it later" — which in practice means never.

> "If an agent can reach a production database simply because it is already inside a trusted network, the architecture has repeated an old networking mistake with a much more capable actor."

The closing line and the best formulation of the whole piece: network membership is not permission, and an agent is the worst possible principal to learn this lesson on.

## Key Themes

- **#concept** — Identity over location: connectivity granted to authenticated identities (user, group, device, policy attached to the connection), never to IP addresses or subnet membership
- **#pattern** — Separation of authorization from network topology: central policy, edge enforcement, point-to-point encrypted connections with relays only as backup
- **#pattern** — Agents as first-class workloads: own identity, revocable access policy, network isolation to exactly the nodes/services/data needed, least privilege from day one rather than "broad first, tighten later"
- **#tool** — Identity-aware gateway between developers and model providers: check who is calling, control which models they may use, hold provider credentials in one place
- **#concept** — Defensive symmetry: the same AI that automates attacker reconnaissance should automate supply-chain checks, environment audits, and build/deploy controls

## Critical Analysis

**What's strong is the failure-mode analogy.** The VPN comparison does real work: everyone in infrastructure already knows how "give the contractor access to everything" ends, and Kukulinski correctly identifies that AI adoption is recreating it under time pressure ("we just need to get this AI thing going"). The prescription — agent as separate workload, own identity, revocable policy, isolated network slice — is the correct default, and it arrives with the practitioner's detail the framework documents lack: pod IP churn, laptop posture, IdP group propagation.

**What's thin is everything an interview can't carry.** This is a vendor explaining the problem its product solves, and the piece never touches the genuinely hard parts: how an agent *gets* an identity (workload identity issuance, per-task credentials, key lifecycle for processes that spawn and die by the minute), whose identity the agent acts under when it works for a user, or what changes mid-task when an agent's context shifts. Identity answers "who is connecting" — it says nothing about "what is the agent being told to do," which is where prompt injection lives. An agent with a perfectly scoped identity can still be hijacked into chaining its narrow grants into an exfiltration path; that is the runtime problem [[The Agent Access Model]] exists for, and this piece is silent on it. The identity-aware gateway paragraph is likewise a product paragraph wearing an architecture paragraph's clothes.

**Where it sits in the wiki:** it is the networking-layer complement to the access-control frameworks — weakest alone, useful as the substrate those frameworks assume. [[Least Privilege for AI Agents — Identity, Access, and Tool Binding]] prescribes dedicated agent identities at design time; Kukulinski supplies the network-side reason why that pillar matters and what skipping it looks like. [[Zero Trust for AI Agents]] declares that only "network paths that do not exist" survive agentic attackers; this interview describes the connectivity model that makes paths not exist by default. And [[The Agent Access Model]] calls for enforcement split across harness and network — Kukulinski's "policy governed centrally, enforcement at the edge" is the network half of that pair, from a company whose whole product is that half.

One genuinely complicating footnote: Tailscale's own [[Tailcat]] experiment deletes the identity and control plane entirely, collapsing identity into a bearer-capability address. The same company therefore ships both the thesis (identity is the control layer) and its reductio (an address that *is* the credential, no identity system at all). The interview never acknowledges that the bearer-capability model is exactly how its agents' workloads would most easily authenticate — which is either a contradiction or the next article.

## Related

- [[Least Privilege for AI Agents — Identity, Access, and Tool Binding]] — strengthens: Microsoft's dedicated-agent-principal pillar gets its practitioner confirmation and its sharpest motivating anecdote ("give AI access to the whole network") from the networking side
- [[Zero Trust for AI Agents]] — instances: the "network paths that do not exist" hard barrier made concrete as isolated networks connected only when a resource must be shared, with edge-enforced policy
- [[The Agent Access Model]] — complements and complicates: AAM's network-enforcement boundary described from the vendor that builds it, while leaving AAM's runtime questions (mid-task narrowing, delegation) untouched
- [[Tailcat]] — complicates: Tailscale's data-plane experiment strips the identity layer this interview insists is the control layer; both agree on direct encrypted point-to-point connections with relays as backup

---
*Sources: [[raw/ai-agents-need-their-own-identity-and-least-privilege-access-11727]], [[summary/ai-agents-need-their-own-identity-and-least-privilege-access-11727]]*
*Last updated: 2026-09-13*
