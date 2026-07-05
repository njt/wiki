# 21 Years and Counting of Eight Fallacies of Distributed Computing

A canonical list of the lies developers tell themselves about networks, born at Sun Microsystems from Bill Joy, Tom Lyon, Peter Deutsch, and James Gosling. George Michaelson revisits each fallacy against the modern Internet and finds none have softened — if anything, the shift from single-provider telecom to multi-provider, multi-intermediary routed networks has made them sharper. The list is aimed at network software developers, but the fallacies apply with equal force to anyone building agent systems that cross network boundaries.

## The Eight Fallacies

1. **The network is reliable** — It's "probably broken somewhere, for some users, at all times." IP doesn't guarantee delivery; TCP and QUIC paper over it with retransmission, but the paper is thin.
2. **Latency is zero** — The speed of light in fibre is slower than in vacuum. Jitter (delay variability) wrecks gaming and streaming. Buffering and FEC compensate but don't eliminate.
3. **Bandwidth is infinite** — Finite bandwidth creates queuing, which creates delay and jitter, which under load becomes packet loss. Your phone does 400 Mbit/s; your five-year-old router caps at 100.
4. **The network is secure** — Networks now cross multiple providers and intermediaries. Even encrypted packets leak through traffic analysis. "Never rely on it below your HTTPS or TLS connections to hide you."
5. **Topology doesn't change** — Phones switch towers, providers reroute traffic. VRRP, CARP, BGP, and Multipath TCP mitigate but aren't free.
6. **There is one administrator** — "Sometimes, it feels as if there isn't even a single administrator." The person you talk to is rarely the person making changes.
7. **Transport cost is zero** — Sending a movie via SMS would cost thousands of dollars. "Just because cost is not exposed directly in a protocol does not mean it doesn't exist."
8. **The network is homogeneous** — BGP mistakes AS path length for cost. Wi-Fi and Ethernet compete differently for the same air. IP masks the differences between local and remote, slow and fast.

## Key Quotes

> "We still seem curiously unable to let go of long-held beliefs."

Michaelson's opener. Twenty-one years after the list crystallized, and developers still write code that assumes the network won't drop packets. This isn't ignorance — it's the gravitational pull of the local-machine mental model. The same force that makes agent developers assume API calls always succeed.

> "The network is probably broken somewhere, for some users, at all times."

The most quotable formulation of Fallacy #1. Not "the network might break" — it IS broken, right now, for someone. The question is whether your system notices or absorbs it. This is the operational reality that [[Queues Don't Fix Overload]] builds on: systems fail gracefully when they acknowledge constraints, catastrophically when they buffer past them.

> "Just because cost is not exposed directly in a protocol does not mean it doesn't exist."

Fallacy #7's sharpest line. The parallel to agent systems is exact: tokens aren't free, API calls aren't free, context windows aren't infinite. Every abstraction layer hides cost; mature engineering surfaces it. This is the same discipline that separates vibes coding from [[Agent Coding Workflow|compound engineering]].

> "IP masks many of the nuances between local and remote, slow and fast."

Fallacy #8's insight. The Internet Protocol's genius — making everything look like a flat address space — is also its trap. Local function calls and remote API calls look the same until latency kills you. Agent frameworks that treat tool calls as uniformly cheap make this exact mistake.

## Key Themes

- #concept — The fallacies as a diagnostic checklist: when a distributed system fails, which fallacy did you forget?
- #pattern — Buffering and retransmission as compensation, not solution. TCP, QUIC, Netflix's FEC — all are mitigations, not cures
- #person — Bill Joy, Tom Lyon, Peter Deutsch, James Gosling at Sun Microsystems. The list is a product of Sun's unique position at the intersection of high-speed graphics, UNIX, and Internet protocols
- #concept — Traffic analysis as the insecurity you can't encrypt away. Even with perfect TLS, metadata leaks

## Critical Analysis

**This is canon, not news — and that's the point.** Michaelson isn't breaking ground; he's performing maintenance on a foundational text. The value is in the refresh: showing that 21 years of Internet evolution (QUIC, BGP, Multipath TCP, home Wi-Fi at 400 Mbit/s) has changed the surface details without touching the underlying truths. The fallacies are physics, not fashion.

**The meta-fallacy is the best part.** Michaelson notes that online discussions "mistakenly refer to Tom Lyon as Dave Lyon" — the list of fallacies about distributed computing is itself subject to a distributed computing failure (data corruption in transit). This is either an exquisite Easter egg or a missed opportunity to make a larger point about how all knowledge degrades across network boundaries, including the knowledge about network boundaries.

**The article reads like an APNIC blog post — which it is.** This is operational wisdom from someone who lives in the infrastructure layer, not academic taxonomy. The examples (SMS movie costs, five-year-old Wi-Fi routers, BGP AS path fallacy) are practical rather than theoretical. The weakness is that it doesn't push into the generative territory: what would a ninth fallacy be? "The network is understandable" — the complexity has outstripped any single operator's mental model. Michaelson hints at this with "sometimes, it feels as if there isn't even a single administrator" but doesn't name it.

**Why this matters for agents.** The [[Distributed Systems]] synthesis page opens with "agent orchestration IS distributed systems." Every fallacy applies directly: agents assume API calls succeed (reliability), ignore round-trip costs (latency), treat context as infinite (bandwidth), trust tool outputs (security), and treat remote services as local function calls (homogeneity). The fallacies are a diagnostic checklist for why agent systems break in production.

## Related Pages

- [[Distributed Systems]] — The synthesis page: agent orchestration reimplements distributed systems primitives, often poorly
- [[Queues Don't Fix Overload]] — Hebert's companion argument: buffering hides constraints until they kill you. The same physics Michaelson describes (finite bandwidth → queuing → catastrophic failure)
- [[Process-Based Concurrency BEAM OTP]] — The BEAM VM was built with these fallacies as first principles: process isolation, supervision trees, "let it crash"
- [[Agent Coding Workflow]] — The maturity spectrum: vibes coding ignores these fallacies; compound engineering accounts for them
- [[All Your Agents Are Going Async]] — HTTP is the wrong transport for agents that outlive connections; Fallacy #1 and #5 in practice

---

*Source: [APNIC Blog](https://blog.apnic.net/2025/12/08/21-years-and-counting-of-eight-fallacies-of-distributed-computing/), George Michaelson, 2025-12-08. Fetched 2026-06-15.*
