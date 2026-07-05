# Nango — Running Untrusted Customer Code at Scale

Nango's three-phase journey isolating untrusted customer code across 150M+ monthly executions: from vm2's in-process sandbox (escaped) through per-customer runners (unfair) to tenant-pinned AWS Lambda functions on Firecracker microVMs — and why the team still debates whether they've built a proper sandbox or just a clever workaround.

---

## Key Quotes

> "An in-process JavaScript sandbox is not a real security boundary. Sharing a process with untrusted code puts you one escape away from a serious incident."

This is the article's foundational claim, and it generalizes well beyond vm2. Any isolation mechanism that shares an OS process, a kernel, or a memory space with untrusted code is a speed bump, not a wall. The 2023 vm2 archive event was the forcing function, but the principle would have asserted itself eventually. [[How We Contain Claude]] reaches the same conclusion from the other direction: "the standard primitives held while our own work around them exposed flaws."

> "Hardware isolation between executions does not help when the same environment serves two customers in turn."

The insight that drove tenant-pinned Lambdas. Lambda's Firecracker microVMs isolate *executions* — each invocation gets its own VM. But Lambda reuses warm environments across invocations, and if two different customers land on the same warm environment, a sandbox escape in one execution could expose the next customer's credentials. This is the kind of threat model depth that separates security engineering from infrastructure engineering. Most teams stop at "we use Lambda, it's isolated." Nango asked "isolated from *whom*?"

> "A resumable run beats a single 24-hour job anyway, since a job that dies at hour 23 and restarts from zero is its own kind of failure."

The product-side solution to Lambda's 15-minute execution cap. Rather than fighting the platform constraint, Nango embraced it: 10-minute caps plus checkpoint-based resumption. This is the same insight behind [[Slate]]'s thread-and-episode architecture and the broader "prefer short, resumable work" pattern that's emerging across agent and integration platforms. Long-running jobs are brittle regardless of platform; resumability is the better primitive.

> "Both views are right — which makes it a real trade-off."

The article's most honest moment. One faction says per-customer Lambdas with warming is just the runner model rebuilt on a different substrate — you've traded Render for AWS but kept the same architecture. The other faction says it's incremental progress: take Lambda's hardware isolation now, work toward a proper sandbox later. McEwan refuses to resolve the debate, and that refusal is the article's intellectual core. Real engineering doesn't end debates; it ships while they continue.

## Key Themes

#sandboxing #isolation #Firecracker #Lambda #multi-tenant #untrusted-code #production-engineering #platform-evolution

**The vm2 lesson: in-process sandboxes are not boundaries.** vm2 was a Node.js sandbox running customer code in the same process as Nango's worker. When sandbox-escape vulnerabilities surfaced in 2023 and the maintainer archived the project, the entire security model collapsed. This is the same pattern [[A Deep Dive on Agent Sandboxes]] identifies: "sandbox policies are code, and code has bugs." The only question is whether the bug is found before or after an incident.

**Per-customer runners as the first real boundary.** Splitting dispatcher from runner over HTTP, with no direct database access — the runner holds only customer code and calls `nango.batchSave()` to a separate persist service. This is defense in depth applied to the application layer. The runner couldn't reach Nango's systems even if the customer code wanted to. But resource fairness and observability were unsolved: one heavy sync could starve other functions, and an OOM kill told you *which runner* died but not *which function* caused it.

**Lambda + Firecracker = isolation with observability.** Each execution gets its own hardware-virtualized microVM with its own kernel. A memory spike now maps to a single function with its own logs. One bad function can't affect others. For a small team supporting hundreds of customers, this "resulted in significant reductions in time spent debugging these issues." The observability win is arguably larger than the isolation win — you can fix what you can see.

**Tenant isolation as the unsolved Lambda problem.** Lambda isolates executions but reuses warm environments. Pinning each customer to their own Lambda functions closes the cross-tenant gap but trades cold starts (9%, up from <1%) for security. The warming workaround — periodic no-op invocations for paid-plan customers — is an honest admission that the platform doesn't quite fit. [[Stockyard]] and [[Navaris]] face the same tension: Firecracker gives strong isolation, but cold starts punish latency-sensitive workloads.

**The WASM path not taken.** Knative and WASM-based runtimes were considered and rejected as "similar to Lambda with far less maturity." Cloudflare Workers' V8 isolates are noted as a weaker boundary than microVMs, and V8 can't safely sandbox arbitrary npm packages — which is what Nango's customers actually run. The WASM sandboxing story is real ([[Component Model 1.0]]) but not yet mature enough for arbitrary Node.js code with third-party dependencies.

## Critical Analysis

This is a valuable article because it's honest about trade-offs in a way most vendor engineering blogs aren't. The internal debate section — where the team openly disagrees about whether per-customer Lambdas are progress or a workaround — is the kind of content that builds trust. Most companies would sand that edge off before publishing.

**What Nango got right.** The evolutionary approach is correct for a small team. vm2 → runners → Lambda is a reasonable path, and each step solved the most pressing problem of its era without over-engineering for problems that hadn't materialized yet. The checkpoint feature is genuinely clever product design — rather than fighting Lambda's 15-minute limit, they made resumability a feature. The security line is clearly drawn: customer code never reaches Nango's systems, and one customer's code never reaches another's. That's a crisp, testable boundary.

**What's undersold.** The move from Temporal to Postgres for scheduling gets one sentence but is probably the most operationally significant decision in the stack. Temporal is a heavy dependency; moving scheduling into Postgres (presumably with something like `pg_cron` or `SKIP LOCKED` queues) is a bet on simplicity that deserves more detail. The article also skips entirely over *how* customer code is deployed — is there a CI pipeline? Image building? Versioning? For a platform that runs untrusted code, the deployment path is itself a security boundary.

**The cold start problem is real and the warming workaround is fragile.** 9% cold starts adding "several seconds" each is material for latency-sensitive action workloads. The warming solution — periodic no-op invocations — is the kind of operational band-aid that works until it doesn't. What happens when AWS reaps a warm environment between warming pings? What's the tail latency distribution? The article acknowledges the workaround without quantifying its reliability.

**Comparison to the agent sandboxing landscape.** Nango's journey parallels the broader agent sandboxing evolution documented in [[Security and Sandboxing]]. The in-process sandbox phase (vm2) maps to the "prompt-level pleading" era of agent security. The runner phase maps to container-based isolation ([[yolo-cage]], [[cco]]). The Lambda phase maps to microVM-based isolation ([[OpenSandbox]], [[Navaris]], [[Stockyard]]). The tenant-pinning insight — that warm environment reuse breaks isolation between tenants — applies directly to multi-tenant agent platforms. If you're running customer agents on a shared [[OpenSandbox]] cluster, you have the same problem Nango found in Lambda.

**The real lesson isn't about Lambda.** It's that isolation is a property of the *system*, not the *substrate*. Lambda gives you hardware-virtualized microVMs, which is excellent. But if you don't also control environment reuse (tenant pinning), credential scope (no DB access from runners), and execution duration (checkpoints + caps), you haven't actually isolated anything. The substrate matters less than the architecture built on top of it. [[How We Contain Claude]] makes the same point from the other side: "the standard primitives held while our own work around them exposed flaws." The primitives (Firecracker, gVisor, Seatbelt) are sound. The architecture around them is where incidents happen.

---

*Sources: [[summary/how-nango-runs-untrusted-customer-code-at-scale]]*
*Last updated: 2026-07-05*
