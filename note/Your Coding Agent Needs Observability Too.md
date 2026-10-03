# Your Coding Agent Needs Observability Too

Mitchel Sellers argues that teams adopting coding agents are running an unobservable production workflow: they see adoption numbers but cannot explain what happened inside a session that was slow, expensive, or wrong. GitHub's OpenTelemetry support for Copilot (announced September 22, 2026) makes execution telemetry — traces, metrics, events — available, and Sellers lays out a conservative rollout: content capture off by default, a small pilot, dashboards built from named operational questions, and telemetry treated as a complement to guardrails, never a substitute.

---

The opening analogy does the work: nobody would run a production workflow without logs, metrics, and traces, "yet that is effectively how many teams are adopting AI coding agents." The observable state today is session started, session ended, dollars billed — enough to know GitHub charged you, not enough to know why an agent repeatedly called the same tool or steered into a dead end.

The load-bearing distinction is **adoption metrics vs. execution telemetry**. Active-user counts and suggestion-acceptance rates are management questions. A trace that connects model calls, tool invocations, failures, and direction changes is a troubleshooting question. He maps it onto the familiar monitoring case: monthly request count vs. distributed trace explaining why one request took eight seconds. GitHub's export covers the classic three signals — traces (session steps), metrics (token usage over time), events (accept/reject moments).

> "This is not about watching individual developers. It is about understanding a new execution system well enough to improve it responsibly."

That sentence is doing quiet political work. Enterprise observability of developer activity is one spreadsheet away from surveillance, and Sellers repeatedly pushes against it: "Avoid turning developer activity into a simplistic leaderboard," and raw counts "do not establish productivity or quality." A developer in an unfamiliar regulated codebase will look worse in the telemetry than one doing repetitive work in a mature repo — the signal confounds task difficulty with operator skill.

His operational cautions are the strongest part of the piece. Content capture (`captureContent` in the managed-settings schema) is off by default for good reason — prompts and tool arguments carry source code, customer data, and secrets — and he says to lock that setting so users can't flip it. He then goes further than the vendor does: **metadata alone is sensitive**, revealing usernames, repository activity, timing, and working patterns, so access control, retention, and audit expectations must be defined before collection begins. He also warns against casually committing a collector token, and mentions using Seq as his OTLP endpoint directly.

The rollout plan is an engineering experiment, not a deployment: small informed pilot, content capture off, documented purpose and retention, two or three operational questions answered, results reviewed *with the developers* before expanding. The closing distinction is the thesis in one line: observability detects and investigates behavior; guardrails limit what the system is allowed to do. Mature adoption needs both.

## Take

A solid, practitioner-grade framing of an important shift: agent sessions becoming observable systems via standard OTel plumbing rather than vendor dashboards. Its real contribution is refusing both failure modes — the surveillance leaderboard and the "we have adoption numbers, ship it" complacency. It is weaker on the specifics (the "what success means" questions are listed but not worked through with real data), and it takes GitHub's announcement largely at face value. But as a checklist for the first week of agent telemetry in an enterprise, it's hard to beat: content capture off and locked, credentials managed, retention defined, questions chosen before dashboards, and a hard separation between metrics-for-managers and traces-for-engineers.

---

Related pages this source strengthens or nuances:

- [[What a Useful AI Trace Should Actually Contain]] — Sellers supplies the enterprise-policy and rollout layer that Iliev's trace-content walkthrough lacks; together they cover what goes in the trace and how to govern it.
- [[Your Agent Should Run Its Own Observability Stack]] — Kin Lane wants agents to run observability inside their own boundary; Sellers shows the organization-side mirror of the same OTel play, on the Copilot/managed-settings path.
- [[Wide Events vs. Three Pillars — AI Observability Costs]] — Sellers cheerfully adopts the three-signal framing (traces, metrics, events) that Honeycomb's piece argues is a cost trap for agent telemetry; a direct tension worth recording.
- [[The Three Pillars of Observability]] — the historical frame for exactly the three signal categories GitHub now exports for agent sessions.

---
*Sources: [[raw/your-coding-agent-needs-observability-too]], [[summary/your-coding-agent-needs-observability-too]]*
*Last updated: 2026-10-03*
