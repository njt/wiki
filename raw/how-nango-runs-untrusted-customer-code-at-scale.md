---
url: https://nango.dev/blog/how-nango-runs-untrusted-customer-code-at-scale/
title: "How Nango runs untrusted customer code at scale"
author: Ross McEwan
date_fetched: 2026-07-05
date_published: 2026-06-08
---

# How Nango runs untrusted customer code at scale

Nango is a code-first platform for building product API integrations. Customers connect apps to Salesforce, Google Calendar, Slack, and hundreds of other APIs. Much of the integration code is written by customers and deployed to Nango, making it untrusted. The company runs over 150 million of these functions per month across varied workload patterns.

## Requirements for the Code Runtime

Three distinct workloads:

- **Actions** (on-demand calls): Run for a user or agent. Fast start/finish; cold starts hurt.
- **Syncs** (long-running jobs): Replicate data in background, sometimes hours across millions of records. Need resumable execution.
- **Webhooks** (bursty events): Arrive in unpredictable spikes. Must absorb sudden floods.

Four additional security/scaling requirements for untrusted code:
- Isolation from Nango's systems (no access to DB, secrets, internal network)
- Isolation between tenants (no cross-customer interference or data access)
- Isolation between executions (jobs shouldn't compete for resources)
- Cost and elasticity (no paying for idle compute; auto-scaling)

## Phase 1: In-Process Sandbox (vm2)

Initially, Nango ran customer code inside vm2, a Node.js sandbox, in the same process as the worker. The customer function executed alongside Nango's own code.

**What went wrong:** In 2023, vm2's maintainer temporarily archived the project after a series of sandbox-escape vulnerabilities. Code inside the sandbox could reach the host.

Key takeaway: An in-process JavaScript sandbox is not a real security boundary. Sharing a process with untrusted code puts you one escape away from a serious incident.

## Phase 2: Isolating Untrusted Code in a Runner

Nango split the system into two parts:

1. **A dispatcher** hands each customer's code to a runner over HTTP
2. **A separate runner** executes it

Each customer got their own long-lived runner, independently scaled (more CPU/memory or extra replicas for heavy accounts). An orchestration layer spins up runners on demand, retires idle ones, and rolls out updates across hundreds. The scheduler behind the dispatcher initially ran on Temporal; Nango later moved it to Postgres.

**Critical design point:** Runners have no direct database access. Functions call an SDK method like `nango.batchSave(...)` which goes to a separate `persist` service over the network. The runner holds only the customer's code and the minimum it needs. Runners ran as separate services on Render.

## Phase 3: Moving to AWS Lambda

By late 2025, the runner model struggled with **resource fairness and observability**. A single heavy job (e.g., replicating millions of records) could starve that customer's other functions. When a runner ran out of memory, it was unclear which of thousands of functions caused it.

**AWS Lambda's advantage:** Each execution runs in its own hardware-virtualized microVM (Firecracker) with its own kernel — "far stronger than a shared process." AWS handles scaling.

Alternatives considered but passed over: Knative and WASM-based runtimes, described as "similar to Lambda with far less maturity."

A memory/CPU issue now pinpoints a single connection's function with its own logs, and one bad function no longer affects others. For a small team supporting hundreds of customers, this "resulted in significant reductions in time spent debugging these issues."

**The Lambda 15-minute problem solved on the product side:**
- A 10-minute cap per run for syncs
- A **checkpoints** feature lets syncs resume across runs instead of relying on one long execution

> A resumable run beats a single 24-hour job anyway, since a job that dies at hour 23 and restarts from zero is its own kind of failure.

## Phase 4: Tenant Isolation on AWS Lambda

**The problem:** Lambda isolates executions but reuses environments to stay fast. AWS may route the next invocation to a warm environment. For Nango, this means two customers could land on the same warm environment. With untrusted code, a sandbox escape could let one customer reach another's credentials in a shared environment. "Hardware isolation between executions does not help when the same environment serves two customers in turn."

**The fix:** Each customer's executions are pinned to their **own Lambda functions**. A warm environment is only ever reused for the same customer. An escape stays contained to what that customer already controls.

**Trade-off:** Per-customer functions cold-start more often. Cold starts went from "well under 1% of invocations to roughly 9%, each adding several seconds." Nango keeps functions on paid plans warm with periodic no-op invocations.

### Internal Debate

**View 1 (pessimistic):** Pinning per customer and keeping functions warm rebuilds the same always-on model as runners but with workarounds around Lambda's limits. The team should instead focus on building a sandbox strong enough that tenant isolation is unnecessary.

**View 2 (pragmatic):** It's sound, incremental progress. Take Lambda's isolation, minimize other changes, close the biggest risk now, keep harder sandboxing work on the roadmap.

The article notes both views are right — "which makes it a real trade-off." A sandbox where escaping gains nothing would eliminate the need for tenant isolation and warming. "But we are not there yet."

## How Others Isolate Untrusted Code

| Service | Approach |
|---|---|
| **E2B** | microVMs |
| **Fly** | microVMs |
| **Modal** | gVisor |
| **Cloudflare Workers** | V8 isolates (weaker boundary) |

The 2025 runc container-escape bugs are cited as a reminder that a shared kernel is not secure. V8 is noted as not a good fit for Nango because customer functions import arbitrary npm packages that V8 cannot safely sandbox.

## Next Steps

The team is tackling tighter isolation down to the individual code function, "so the Lambda boundary wraps a single function, and there is even less to reach if something breaks out."

Nango is open source and hiring.

## Lessons Learned

1. **"Know which boundaries actually stop an attacker"** — an in-process sandbox is escapable, so it shouldn't count as isolation.
2. **"Draw the security line where your product needs it"** — Nango's line: customer code never reaches Nango's systems, and one customer's code never reaches another's.
3. **"Fit the runtime to the workload, and call workarounds what they are"** — acknowledging workarounds keeps them from becoming the architecture.
4. **"Prefer short, resumable work over long-running jobs"** — resumable runs survive failures, work better for rate-limits, and are more practical.
