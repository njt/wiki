# Operating Mode as Runtime State

O'Reilly Radar's contract for exception-aware agents: the temporary operating state an organization grants during an incident — emergency routes, shortened approvals, elevated tool access — should be handed to agents as authoritative runtime input (mode, scope, authority, expiry, status), not inferred from prompts or memory, because otherwise a controlled exception quietly settles into the platform's standard behavior. The essay names this **exception drift** and argues the fix is architectural: governance lives in the runtime, where closure can be validated rather than merely remembered.

---

## Key Quotes

> "Exception drift is what happens when temporary exception behavior outlives its authorized scope, authority, or duration, and emergency accommodations settle into normal execution."

The coinage the whole piece hangs on. Note the drift is "usually quiet: a routing rule that stays reachable, an approval shortcut that survives closure, a tool permission that keeps shaping execution after the triggering condition has passed." No dramatic model failure is required — only a platform with no reliable way to close runtime state.

> "Memory informs execution; operating mode governs it. And when the two disagree, authoritative runtime state wins."

The load-bearing distinction for [[Agent Memory and Context]]-style architectures. Historical traces and retained memory can explain *why* an accommodation once existed; they must never decide whether it is *still authorized*. This is a direct strike against the pattern of agents reconstructing context from accumulated history — the same failure surface compaction and summarization create.

> "Permissions determine who may act, and policies determine how they may act. Operating mode determines whether exception behavior is authorized at all."

The triad that positions the proposal against familiar primitives. Feature flags target behavior by context, RBAC governs principals, tenancy metadata routes requests — none of them answer whether the organization is currently in a state where exception behavior is permitted at all. Operating mode is claimed as a *higher-order* governance constraint sitting above permissions, policies, tools, and escalation paths.

> "Declaring an exception is loud. Retiring one is quiet, especially when the workaround improved throughput or helped the team recover faster. That asymmetry is where drift lives."

The mechanism in one sentence. The exception that *helped* is precisely the one nobody wants to retire — an incident can be closed on paper while emergency routing keeps influencing execution. Enterprises are practiced at the front of the exception lifecycle (declare, authorize) and weak at the retirement end: proving the exception behavior actually disappeared.

> "Emergency behavior exists because the platform enables it, and for no other reason."

The reframing that makes the boundary testable. If the platform treats exception behavior as capability gated behind operating mode, then "a workflow in normal mode should never reach an emergency path" becomes an enforceable runtime property the platform can check at execution time — governance as an architectural invariant, not a policy document.

> "No one has to remember to retire the route; it was bounded by state, and the platform can show it is gone."

The closing scenario's payoff. Closure runs as a check: the platform replays the exception's scope against live routing, approval, tool, and queue configuration and confirms no path still resolves to the emergency behavior. Retirement stops being a human memory task and becomes a verifiable system property.

## Key Themes

#concept #pattern #tool

- **#concept — Exception drift**: temporary accommodations outliving their authorized scope, authority, or duration and hardening into standard behavior. The runtime-state cousin of [[Principal Drift]]'s loss of control — drift through config that never closes rather than review that never happens.
- **#concept — Operating mode as authoritative runtime input**: organizational operating state (normal, incident, recovery, declared exception) delivered alongside identity, tenant, environment, and permissions on every request. Never inferred from prompts, conversation history, or agent memory.
- **#pattern — The runtime contract**: six fields — mode, exception ID, scope (segment/region/workflow), authority, expiry, status — that bind an exception to its justification and make "is this emergency path reachable right now?" a checkable question.
- **#pattern — Validated closure**: end-of-incident replay of the exception's scope against live configuration, proving retirement ahead of manual cleanup. Plus ongoing drift observability: open exception counts, duration, and hardening rate as monitored signals.
- **#pattern — Shared operating state for multi-agent systems**: the exception represented once and read consistently by every collaborating agent — "Fragmented state produces fragmented accountability."
- **#tool — External control plane**: incident management (PagerDuty), change management (ServiceNow), and maintenance-window services as the natural home for authoritative operating state, extended into execution with scope, authority, expiry, and closure semantics.

## Critical Analysis

**The core move is the BeyondCorp trick applied to operational posture, and it is sound.** Google's [[Beyond Zero — Enterprise Security for the AI Era]] made authorization a machine-speed, per-action runtime decision; this essay makes organizational *posture* a runtime input. Both reject the same thing: governance that lives in documents and human memory rather than in state the platform can check. The distinction the essay draws from feature flags is real and worth preserving — a flag targets behavior by context but carries no authority chain, expiry, or closure semantics, while the contract binds an exception to an owner and an end condition. "Governance becomes an enforceable runtime property the platform can check at execution time" is the most falsifiable claim in the piece, and it's the right one.

**The argument that agents raise the stakes is the strongest part.** A traditional exception "stays legible in a runbook or workflow definition"; an agent "can carry the same exception along many paths at once, which makes it harder to find and retire." Agents act — they select tools, trigger workflows, coordinate with other agents — so an accommodation spreads rather than sits. This is a genuinely new failure surface that pre-agent governance literature does not cover, and the essay is right that no dramatic model failure is required for it to bite.

**But the hard engineering is hand-waved.** The essay never says who populates the control plane, how operating mode reaches third-party SaaS that doesn't read it, or what happens when an agent caches mode state mid-request. The example JSON is six fields; the real cost is propagating that state through every tool gateway, workflow engine, and downstream agent — and handling the human who bypasses the gateway entirely. The claim that "none of this demands a new governance model" is optimistic in the specific place it matters: incident-management systems are notoriously bad at exporting clean, machine-readable, expiring state, and that integration is exactly what the whole design stands on.

**It is also governance for benign exceptions only.** Nothing here addresses an adversary who wants the agent to *believe* incident mode is active. The essay's implicit answer — state must be external and authoritative, kept out of prompts and memory — is the right architecture, but it never connects to the injection literature ([[LLM01 Prompt Injection (OWASP)]], [[Bounding the Blast Radius — Prompt Injection Defenses]]), where "convinces the agent it's in an emergency" is already a documented social-engineering pattern. An operating-mode contract is a defense only as long as the channel serving it is trusted; the essay assumes that channel and never names the threat.

**Where it slots in.** This is the third essay in the O'Reilly Radar governance sequence that [[Principal Drift]] anchors: principal drift named the loss of control through review failure; this names a second channel — runtime state that never closes — and both insist the fix is architecture, not process. It nuances [[Beyond Zero — Enterprise Security for the AI Era]] by adding an axis Beyond Zero doesn't have: Beyond Zero answers "what may this accessor do right now?", operating mode answers "is the organization in a state where exception behavior is authorized at all?" — together they sketch a layered authorization stack (permissions → policy → mode). It strengthens [[How We Contain Claude]]'s central lesson — model-layer defenses are probabilistic, environmental controls are deterministic — with a governance twin: "prompt wording stops being the lever" because behavior shifts when system state shifts. And it complicates [[AI Handles Incidents, Engineers Lose Touch with Their Systems]]: Kalache worries AI-assisted incident response atrophies the humans; platform-validated closure automates the postincident review too, and the postincident review is one of the last rituals where responders actually learn their system. Neither piece resolves that tension.

---

*Sources: [[raw/operating-mode-as-runtime-state-a-contract-for-enterprise]], [[summary/operating-mode-as-runtime-state-a-contract-for-enterprise]]*
*Last updated: 2026-09-13*
