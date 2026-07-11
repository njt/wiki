# Cloud Agent Lessons from Cursor

Cursor's engineering team on what it actually takes to build coding agents that run on remote infrastructure instead of your laptop. The core insight: the development environment IS the product, and the engineering challenge is reconstructing everything a local agent inherits for free — tools, configs, credentials, repo state — on ephemeral cloud VMs. The secondary insight: long-running agents need durable execution infrastructure (Temporal), and you must decouple agent logic from both machine state and conversation state.

---

## Key Quotes

> "When building a local coding agent, you inherit the user's existing development environment. It's set up the way they like it, with all their tools, configs, and credentials already in place. For cloud agents, you have to reconstruct the development environment on a remote machine, from scratch."

This is the framing insight. Local agents have a massive unearned advantage — they inherit a working environment. Cloud agents start from zero. The phrase Josh Ma uses for incomplete environments is "a subtle degradation in output quality" — the agent doesn't fail, it just produces worse code. This is harder to detect and harder to fix than explicit errors. It's the same dynamic [[Components of a Coding Agent]] identifies: "a lot of apparent 'model quality' is really context quality." For cloud agents, context quality starts with environment quality.

> "Reliability is a threat vector in and of itself."

Not a secondary concern, not an ops problem — a first-order threat to the product. When your agent can be killed by an EC2 node failure mid-task, reliability isn't about uptime metrics, it's about whether the product works at all. This converges with [[How Hightouch Built Their Long-Running Agent Harness]]'s finding that production agent engineering is "deeply unfussy work" — infrastructure, not magic.

> "Temporal gives us powerful primitives: durable timers, automatically retried activities, and the ability to continue execution even in the face of node failures."

The most specific infrastructure recommendation in the post. Cursor migrated from a fragile work-stealing architecture to Temporal and went from "one 9 to two 9s of reliability." Today Temporal handles 50M+ actions/day across 7M+ workflows. This is the same bet [[All Your Agents Are Going Async]] argues for: agents need durable execution that outlives individual connections and machines. Cursor is the largest public proof point.

> "Early on, our agent harness would double-check its work after every task, force a commit, and push. Over time, we realized something counterintuitive: the more capable the model, the more the harness should get out of the way."

This is the maturation arc of an agent platform. The harness starts as training wheels, then becomes a constraint. As models improve, logic migrates from the harness (where it's fixed and brittle) into the agent's tools (where it's flexible and contextual). This is the inverse of [[Harness Engineering (OpenAI)]]'s thesis — OpenAI argues for *more* harness, Cursor argues for *less*. The resolution is that both are right at different stages: you need the harness to get to reliability, then you need to know when to remove it.

> "The cost of blocking is much higher for cloud agents."

A cloud agent waiting for human input sits idle for hours, burning compute and delaying results. Local agents can afford to block — the human is right there. This single difference cascades through prompt design, tool availability, and autonomy defaults. Cloud agents need prompts that say "figure it out" rather than "ask if unsure."

## Key Themes

### #pattern The Development Environment as Product

The local agent inherits a working environment; the cloud agent must have one built. This isn't just about getting things to compile — incomplete environments degrade output quality silently. Cursor's response is "enterprise IT for agents": VM hibernation, checkpoint/restore, secret redaction, network policies, credential management. This is the infrastructure layer that makes cloud agents viable, and it's invisible when done right. [[Building Agents That Don't Break Themselves]] describes the same pattern from the other direction: disposable nested sandboxes with copy-on-write checkpointing as a reflex.

### #tool Temporal as Agent Infrastructure

Temporal isn't just a workflow engine — it's becoming the standard durable execution layer for coding agents. Cursor runs 50M+ actions/day on it. The "Lessons from Building Cursor" ByteByteGo talk also name-checks Temporal and Restate. The architectural pattern is clear: agents are long-running processes that need retry, scheduling, and failure durability — exactly what Temporal was designed for. [[Apache Burr]] is the alternative for teams that want explicit state machines rather than Temporal's workflow model.

### #pattern Decoupling Agent, Machine, and State

The three-way split (agent loop in Temporal, machine state independent, conversation state as append-only stream) is the architectural insight that makes everything else possible. Without it, a machine failure kills the conversation. With it, the agent's work survives infrastructure churn and the user's view stays consistent. This is event sourcing applied to agents — the append-only log as the source of truth, with clients rewinding and replaying as needed. [[The Log is the Agent]] describes the same pattern as a general principle.

### #concept Harness Retreat as Model Progress

The harness should shrink as models improve. Early on, the harness double-checks everything; later, it gets out of the way. This is a maturity model for agent platforms, not a design preference. It explains why different teams give opposite advice about harness complexity — they're at different points on the same curve. [[Lessons from Building Cursor]] reinforces this from the model-training side: RL teaches behaviors that prompts can't. When the model internalizes verification, the harness doesn't need to enforce it.

### #pattern Self-Healing Environments

The frontier: agents that debug their own environments. Missing secrets, blocked network access, wrong tool versions — the agent should detect and fix these, not fail silently. Cursor's "autoinstall" concept points toward environments that are legible to the agents running inside them. This is the logical endpoint of "the development environment is the product" — an environment that can explain itself to the agent and repair itself when broken. [[Agent-Native Architectures (Every)]]'s "parity principle" (agents get the same interfaces humans do) points in the same direction, but self-healing goes further: agents get *better* interfaces than humans, because they can act on diagnostic information automatically.

## Critical Analysis

This is one of the most useful engineering posts from a frontier coding-agent company. It's specific where most posts are vague — named infrastructure (Temporal), concrete metrics (one 9 to two 9s, 50M actions/day, 40% of PRs), honest about failures. The contrast with the "Lessons from Building Cursor" ByteByteGo talk is instructive: that talk was vision and conviction; this post is engineering and architecture. Both are valuable, but this one is actionable.

The "harness retreat" thesis is the most provocative claim and the least defended. Cursor says they moved logic out of the harness as models improved, but they don't say *which* logic, *when*, or what the failure mode was when they moved too early. This matters because the advice is inherently risky — remove safety rails from a system that's still learning — and the post doesn't help you decide when it's safe. [[Harness Engineering (OpenAI)]]'s team went the opposite direction, adding more structural constraints as they scaled. Is the difference model capability (Cursor's custom RL-trained models vs. OpenAI's Codex), task type, or philosophy? The post doesn't say.

The "subtle degradation" problem is underappreciated and probably the most important reliability challenge for cloud agents. Silent failures are worse than loud ones because they compound — the agent produces slightly worse code, which means slightly more bugs, which means slightly more cleanup work, all without anyone noticing the root cause. This is the cloud-agent equivalent of [[Constraint Decay]]'s finding that agents lose ~30pp assertion pass rate under structural constraints — the failure mode is gradual degradation, not sudden collapse.

The Temporal migration is the most convincing infrastructure recommendation in the post. "One 9" to "two 9s" maps to ~90% to ~99%+ reliability — that's going from "fails every 10th task" to "fails every 100th task." For a product where tasks take minutes to hours, that's the difference between usable and unusable. The evolution from eternal workflows to short single-task workflows is the kind of detail you only learn in production — it's not in Temporal's docs, it's earned wisdom.

What's conspicuously absent: cost. 50M actions/day on Temporal isn't free. VM hibernation and checkpoint/restore infrastructure isn't free. "Enterprise IT for agents" isn't free. The post describes infrastructure without economics, which makes it hard to evaluate whether these patterns apply outside a well-funded startup. A solo developer can't run Temporal at Cursor's scale. The post would be stronger with even rough order-of-magnitude cost estimates.

The self-healing environments section is a teaser, not an argument. It gestures at "autoinstall" without explaining what it is or whether it works. This is the most important unsolved problem in cloud agents — the gap between "environment is set up perfectly" and "agent figures it out from scratch" is the gap between a demo and a product — and the post acknowledges the problem without solving it.

Still: essential reading for anyone building agent infrastructure. The three-way decoupling (agent/machine/conversation), the Temporal migration story, and the harness-retreat maturity model are patterns that will show up in every cloud agent platform. This post documents them first.

---

*Sources: [[raw/cloud-agent-lessons]]*
*Last updated: 2026-07-11*
