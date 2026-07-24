# AgentPulse

An open-source drift detection and investigation tool for multi-agent systems: instrumentation, local capture (SQLite, no cloud), baseline comparison across runs, and root-cause-led investigation. MIT-licensed by Prove AI, with Claude Code MCP integration.

---

AgentPulse addresses a problem that every production multi-agent system eventually hits: **outcomes degrade, and you don't know which agent broke or why.** Traces show you what happened; AgentPulse shows you where to investigate.

It auto-instruments OpenAI, Anthropic, LangChain, and AutoGen with two lines of code, stores LLM calls, agent turns, tool calls, and handoffs in a local SQLite file, then compares behavior against a baseline to flag drift in agents, handoffs, and routes.

> *"Traces tell you what happened. AgentPulse shows you where to investigate."*

## Investigation Over Trace Viewing

The core design bet is that **root-causing multi-agent failures requires structured investigation, not another trace viewer.** When outcomes drop, AgentPulse walks the agent graph upstream, component by component, looking for the precise node where inputs are stable but output drifted. It then correlates that with any prompt, model, or tool changes that landed nearby.

The methodology is captured in four rules:

1. **Baseline everything** — every metric, per agent, handoff, and route
2. **Don't chase noise** — only open investigations on sustained outcome breaches, not single spikes
3. **Walk upstream** — follow the graph until inputs are stable but output isn't
4. **Correlate the changes** — prompt, model, and tool changes are the usual suspects

This is essentially **differential diagnosis for agent systems**: systematic elimination rather than intuition-driven debugging. It's the same mental model doctors use, formalized as a tool.

## Local-First, Conversation-Ready

Two architectural choices stand out:

**Everything runs locally.** Capture stores to SQLite per project, no cloud account or hosted backend required. This is the right call for debugging tools — you don't want your observability to go down when your infrastructure does, and you don't want sensitive agent traces leaving your machine.

**MCP server ships in the box.** Claude Code and Claude Desktop can query drift findings directly: "what drifted today?" returns a root-cause-led investigation card with severity, confidence, and next steps. Three skills (`get_todays_finding`, `get_version_comparison`, `get_next_check_steps`) make the tool queryable conversationally rather than through a dashboard alone.

## Opinionated Analysis

**What's genuinely new.** The graph-walking methodology — follow drift upstream until inputs stabilize but output doesn't — is a structured algorithm where most teams operate on intuition. Formalizing it as a tool workflow is a real contribution. The change-correlation step (flagging that a prompt + model change landed at run 18) closes the loop from detection to likely cause.

**What's ambitious.** The four rules read like a diagnosis protocol written by someone who's debugged enough multi-agent failures to know the patterns. Rule #3 ("stop where inputs are stable but output drifted") is the sharpest — it's exactly how you'd teach a junior engineer to triage, now encoded in software.

**What's unproven.** The page is a product landing page and reference implementation, not a field report. No production metrics, no scale numbers, no failure modes of the tool itself. The "open experiment" framing is honest about this — they're shipping a reference implementation to learn how teams investigate agent failures, not claiming to have solved it. The real test is whether the drift detection signals have enough signal-to-noise to avoid drowning teams in false positives.

**The gap.** There's no mention of evaluation or ground truth. Detecting "drift" requires knowing what correct behavior looks like, and for multi-agent systems that's often the hardest part. The tool can tell you the writer's output changed, but it can't tell you whether the new output is *wrong* — that judgment still requires human evaluation or a separate eval harness.

## Connections

This sits at the intersection of several threads in the wiki. It's a [[Guardrails and Feedback Loops|guardrail tool]] for multi-agent systems — the monitoring layer that tells you when to investigate. The investigation methodology echoes [[Ways of Checking]]: systematic verification rather than re-running the same check. The graph-walking approach is a lightweight version of what [[DDB — Source-Level Interactive Debugging for Distributed Applications|DDB]] does for distributed systems.

The local-first capture design connects to the broader pattern of tools that keep agents' operational data on your own infrastructure ([[Building Agents That Don't Break Themselves]], [[The Log is the Agent]]). The MCP integration makes it part of the [[Components of a Coding Agent|growing ecosystem]] of tools that treat agents as first-class systems to be operated, not just developed.

For teams running multi-agent pipelines ([[Agent Orchestration]], [[Agent Swarm Model Economics]]), AgentPulse fills the observability gap that raw traces and logs structurally cannot — structured investigation over raw data.

The "open experiment" stance — shipping a reference implementation to learn how people actually debug agent failures — is the right instinct. The tooling around agent reliability is still being invented, and the smart play is to instrument, observe, and iterate rather than pretending we have the answers.

---

*Sources: [[raw/agentpulse]]*
*Last updated: 2026-07-25*
