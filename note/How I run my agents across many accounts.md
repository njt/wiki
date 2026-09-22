# How I run my agents across many accounts

This is a field report of one person's complete agent software factory — "Mujin," named after the Japanese for a lights-out factory — spanning issue grooming, parallel producers, a multi-account seat pool, an independent landing gate, and post-commit review. It matters because it is one of the few write-ups that describes the *whole* pipeline at personal scale, including the parts that don't work yet.

---

## The argument in one paragraph

The source claims that a single person can run a lights-out agent production line across many repos and many vendor subscription seats, provided three invariants hold: no producer lands its own code (main has exactly one writer), a human is the only path to the label that makes an issue executable, and every agent's liveness is observable through state files rather than screen-scraping. If the author is right, the binding constraint on personal agent throughput is not model capability but seat metering, session observability, and landing discipline — and the multi-seat pool is a legitimate, terms-compliant answer to the first of those. This is falsifiable: if the seat-pool relay shape is actually a terms violation, or if the night lane's 90-minute cap and two-failure stop mean the unattended lane contributes trivially little, the system is an elaborate toy around an attended core.

---

## Key quotes

> The organizing principle is that no producer lands its own code. Main has exactly one writer, and a bounce is the normal retry path.

This is the whole merge-gate architecture in two sentences. Compare-and-swap pushes, a rebased train under one gate, and "there is no retry command" turn landing from a courtesy into a protocol — the producer fixes its branch and resubmits, so blame never blurs.

> The safety principle is that a human is the only path to the label that makes an issue executable, and the grooming lane that proposes that label cannot write it.

A separation-of-powers design: the agent that proposes work can never authorize it. The auto-promoter's cap (three low-risk cards per run, only ones the reviewer passed) is the pressure valve, and everything with a real tradeoff surfaces as one Telegram message per decision.

> After I once approved a card and eight decisions without understanding any of them, every message now has to open with six fixed lines: the ask as a question, what happens if yes, what happens if no, a recommendation, why this needs me specifically, and how to undo it.

The most honest sentence in the piece. The author rubber-stamped nine decisions, noticed, and responded by making the *interface* enforce comprehension — a validator refuses messages that break the shape. That is human-factors engineering applied to human-in-the-loop approval.

> The live-session term exists because one morning a burst of launches all read the same five-minute-old snapshot and piled a dozen Fable sessions onto one seat while other seats sat at forty percent with nothing on them.

Every load-balancing heuristic here is scar tissue. The "coldest seat" score is worse-of-two-meters plus ten points per live session plus an LRU tiebreak in microseconds — each term traceable to a specific failure, which is what distinguishes this from a design essay.

> This came out of a coordinator run that sent 287 steering messages into a builder mid-thought and polled 1,629 times, and landed nothing.

The numbers make the point better than any argument: an eager coordinator that cannot tell "working" from "wedged" is worse than no coordinator. The fix — three levers, escalating from written directive to stop label to direct conversation, with the send path refusing to paste into a working builder — is a communication protocol, not a UI tweak.

---

## Critical analysis

The non-obvious contribution is the seat pool. Most published agent-factory architectures assume API spend or a single seat; this author treats personal subscription seats as a schedulable resource, with meter polling, headroom gates per model lane, and a sweeper that migrates stalled sessions between seats. The terms-of-service section is unusually careful — the pool is a launcher that execs the vendor's binary, never a socket-listening relay — and the argument that the relay shape (not the multi-seat fact) is what gets accounts terminated is a claim worth tracking. It is also the part a reader should be most skeptical of: "I only ever run the official harnesses" is self-reported compliance, and the economics of buying multiple personal subscriptions to feed a factory sit uncomfortably close to the line the author draws.

The second strength is that the failure modes are load-bearing. The liveness watcher exists because five dispatched builds sat idle for eight to sixteen hours. The hook-state file exists because TUIs redraw their clock while idle, making pane activity worthless as a signal. The "session is the identity, the pane is just where it is attached" reframing is a genuine architectural insight for anyone running agents in tmux.

What is weak: the write-up is a single point of failure — one person, one Mac, one Telegram account — and the author admits the sweeper misses a whole tmux server and some banner shapes. The night lane's real yield is never stated; five issues per night with a two-failure stop could be a rounding error next to the attended sessions. And the sensitivity routing to GLM and Kimi, while carefully gated, is asserted rather than audited — "their output is never used as fact without verification" is a policy, not a mechanism.

What is left out: cost. There is no token or dollar accounting anywhere, despite the system's whole premise being meter management. Nor is there any comparison of throughput before and after Mujin — we never learn whether the lights-out factory actually out-produces one person with Claude Code and good habits.

---

## Related

- [[Fleet Supervisor (sermakarevich)]] — Strengthens the same supervisor pattern from the other direction: both split a dumb lifecycle manager (claim, spawn, reap, retry) from intelligent coder backends, but Mujin adds the seat-metering and liveness layers the Fleet Supervisor leaves to the operator.
- [[How to Build an AI Software Factory]] — Complicates it: Firecrawl's five-stage control plane (intake, isolation, tools, verification, merge gate) maps almost one-to-one onto Mujin's stages, but this source shows the same shape built by one person on personal subscriptions rather than an org platform, undercutting the "factory needs infrastructure" framing.
- [[108 PRs in Eight Days — Accidentally Discovering Loop Engineering]] — Nuances Ellich's finding that the human becomes the bottleneck at both ends of the pipeline: Mujin attacks exactly that with the Niwashi grooming lane and six-line decision messages, an answer to the spec-work squeeze she leaves open.
- [[no-mistakes]] — Strengthens the shared conviction that no producer should land its own code: no-mistakes gates a single push through a validation pipeline, while Mujin generalizes it to a single-writer lander with bounce-as-retry across eight repos.

---
*Sources: [[raw/how-i-run-my-agents-across-many-accounts]], [[summary/how-i-run-my-agents-across-many-accounts]]*
