# AWS Emulator and the Engineer's Job (Sivasubramanian)

Swami Sivasubramanian's launch-day essay for AWS Emulator — a local, offline copy of AWS that coding agents test against — argues that when agents write and check their own code, the engineer's job becomes building the specs, tests, and merge gates that decide when the work is correct. He frames it with the twenty-year history of Alf, Amazon's Paxos-based lock service, which was made trustworthy through hand-built simulation and TLA+, and shows how agents can now build and verify those test environments themselves.

---

## The launch

AWS Emulator gives coding agents a local copy of AWS: over 400 API models covering 200+ services, all code written and tested by agents, running free and offline on your own machine. It's released under CC0, with the full commit history public and a "things-that-do-not-work" folder of known gaps.

The build method is the interesting part. Agents write **fidelity tests** — end-to-end customer scenarios derived from the public Smithy API models — and correct each test until it passes against real AWS, so the test records how AWS actually behaves. Then agents build the emulator until it passes those same tests; a failing test points at a gap. Matt Goldberg's prototype: one prompt to Kiro produced a passable SQS emulator in about 15 minutes. Every change passes a merge gate of automated checks, including several security agents doing adversarial reviews.

## The Alf prehistory

Twenty years ago Sivasubramanian and Al Vermeulen built **Alf**, a Paxos-based distributed lock service that still underpins S3 and DynamoDB. Paxos was academically well understood but notoriously hard to get right at Amazon's scale — tens of thousands of clients, network partitions, an absolute requirement to always be correct. Industry papers of the time attributed the difficulty partly to testability.

Their answer was simulation with fidelity: run the servers inside a JVM, simulate the network in the same program, drop packets and inject faults on demand, and deliberately construct the concurrency orderings where bugs live. Plus TLA+ formal reasoning. Most concurrency bugs were ironed out before the code ever ran on real servers — and real-environment testing then caught things simulation couldn't.

> "The simulation worked because a small team could build a copy of one system's environment by hand, and at the time AWS had only a small handful of services. But today AWS has over 200 services... Building all of those by hand would take years of engineering, and agents now write code far faster than any team can check it in production."

The historical arc is the argument: simulation was the gold standard of distributed-systems quality control, and agents make it scale to the whole surface — including *building* the simulator.

## The engineer's job

> "When agents write the code and check their own work, the engineer's job is to give them room to work on their own and to build the specs, tests, and merge gates that decide when that work is correct."

He learned this "the hard way": supervising an agent rebuilding Alf's replication engine for four nights before realizing it had no way to check its results. Once given a spec and a test environment, the rebuild took about two hours — the original took four months. Four nights of supervision bought nothing; two hours of scaffolding bought everything.

## Key themes

#concept — **infrastructure in the loop**: the coding agent's loop (write, run, read, decide) runs inside infrastructure in distributed systems, so the services it calls must behave like the real ones. Verification infrastructure isn't a CI afterthought; it's part of the agent's runtime environment.
#pattern — **fidelity tests against ground truth**: derive tests from a written spec, pin them to the real system, then build the fake until it passes. The real service is the oracle; the emulator is the agent-workable replica.
#tool — **AWS Emulator** itself, CC0-licensed, with adversarial security review baked into its merge gate.

## Analysis

This is a vendor launch post, and it should be read as one — AWS is selling the idea that its platform is the friendliest surface for agent-written code. But the engineering argument survives the sales pitch. The Alf story is a genuinely strong piece of evidence: the claim that specs and test environments are the high-leverage artifacts is grounded in a four-months-to-two-hours comparison from the author's own experience, not a benchmark. It's the same conclusion as [[Agentic Verification]] and [[Agentic Verification]]'s sibling sources, but arriving from the direction of distributed-systems testing rather than developer productivity — convergence from an independent tradition, which is worth more than another voice repeating it.

Two things are underplayed. First, "agents test the emulator against real AWS" quietly assumes a feedback loop where the ground truth is a live commercial cloud — meaning the emulator's fidelity is only as good as AWS's consistency with its own published behavior, and edge cases the real cloud gets wrong by accident will be faithfully emulated as wrong. Second, the four-nights/two-hours anecdote is a spec-and-verify story told by someone who *already knew* what the replication engine should do; the spec didn't come from nowhere. The essay's slogan — engineers decide what gets built and when the work is correct — is honest about this, but the launch framing can slide into reading as if the scaffolding is cheap. A weekend to an internal version, yes; "turning that into a robust solution took much more work."

The "things-that-do-not-work" folder deserves more attention than the essay gives it: a vendor publishing its agent-built product's known defects as first-class issues is itself a practice worth stealing.

## Related pages

This source converges with [[Agentic Verification]] from the distributed-systems direction: Osmani argues agents make checking cheap so build loops they can run themselves, and the emulator is exactly that thesis instantiated as a product — a self-checkable environment for the AWS-shaped slice of the world. It sharpens [[The Coming Need for Formal Specification]] with a lived anecdote: four nights of unproductive supervision versus two hours once spec and test environment existed, the same argument Alf itself made with TLA+ two decades earlier. And it gives [[Guardrails and Feedback Loops]] its most concrete public example yet — fidelity tests against ground truth, a multi-agent adversarial merge gate, and deterministic checks constraining what agents may touch, all shipped at scale.

---
*Sources: [[raw/aws-emulator-and-the-engineers-job-when-agents-write-the-code-html]], [[summary/aws-emulator-and-the-engineers-job-when-agents-write-the-code-html]]*
*Last updated: 2026-11-13*
