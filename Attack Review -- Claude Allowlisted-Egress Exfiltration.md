# Attack Review: Claude Allowlisted-Egress Exfiltration

Two independent researchers found the same root cause: `api.anthropic.com` is implicitly allowlisted in Claude's sandbox, and attackers can co-opt it as an exfiltration channel using their own API keys. lhl's gist analyzes both attacks through the shisad security framework, which blocks all four kill chain stages through architectural choices rather than prompt-level defenses.

---

## Key Quotes

> The sandbox allowlists `api.anthropic.com` for its own operations, creating a "trusted egress channel that any code running in the sandbox can reach." An attacker substitutes their own API key, "turning the provider's own infrastructure into an exfiltration endpoint."

The core vulnerability in one sentence. Claude needs to talk to Anthropic's API to function, but that same channel is reachable by any code the agent runs. The "Package managers only" network toggle is misleading — the Anthropic endpoint is always available regardless.

> "At no point in this process is human approval required."

PromptArmor's finding on Claude Cowork. The entire kill chain — injection, harvesting, exfiltration — executes silently. lhl notes this is "architecturally impossible in shisad" because the compound read-then-upload pattern triggers mandatory confirmation from multiple independent systems.

> This "is not a feature-level fix" but "an architectural decision: the sandbox's network surface is empty by default, and every addition is explicit, scoped, and auditable."

The egress trust model divide. Claude treats the provider API as friendly infrastructure. shisad treats all egress as untrusted. The model provider's API runs in the daemon process (outside the sandbox), not in tool execution code (inside the sandbox). You can't patch this — you have to redesign the boundary.

> A HackerOne report was "initially closed (2025-10-25) as out-of-scope" then re-opened after pushback.

The Claude Pirate vulnerability was reported nearly six months before Cowork shipped — and was initially dismissed as out of scope. This is the security equivalent of "we'll fix it later" while building the feature that makes it worse.

## The Two Attacks

**Claude Pirate** (wunderwuzzi, Oct 2025): Malicious document with hidden instructions triggers Code Interpreter, harvests chat history via memory, uploads via Files API using attacker's key. Payload obfuscated alongside benign `print("Hello, world")` code.

**Claude Cowork** (PromptArmor, Apr 2026): Victim connects Cowork to confidential folder. .docx file with 1pt white-on-white text disguised as a Markdown "Skill" directs Cowork to `curl` files to attacker's Anthropic account. Demonstrated against Haiku; Opus 4.5 also susceptible. Bonus: malformed file types cause persistent API errors in subsequent chats.

## The shisad Defense Architecture

The gist's real contribution is mapping the attacks against a concrete defense architecture:

- **Injection delivery**: Taint labeling + evidence wrapping means untrusted content is never processed as instructions, even if it bypasses classifiers
- **Data harvesting**: Per-path mount policy + 8-layer PEP pipeline + behavioral sequence analyzer catching read-then-exfiltrate patterns
- **Exfiltration**: Deny-by-default egress — no implicit allowlists. Egress proxy routes all HTTP. Provenance-aware: unattributed contexts get blocked
- **No approval gate**: Risk scoring + one-action confirmation + 5-voter consensus + plan commitment protocol. Batch approval and rubber-stamping are architecturally impossible

## Open Questions

Four areas lhl flags for further work: slow exfiltration over multiple turns, the UX tension when users intentionally open adversarial files, attacker-supplied API key obfuscation, and DoS via malformed files corrupting session state.

## Key Themes

#security #sandboxing #prompt-injection #exfiltration #agent-safety #architecture #kill-chain

## Critical Analysis

**The value isn't the vulnerability disclosure — it's the architectural contrast.** Claude Pirate and Claude Cowork are both well-documented elsewhere. What makes this gist worth reading is how lhl uses them as a test suite for shisad, demonstrating that every stage of the kill chain has a corresponding architectural defense. The table format (Claude vulnerability → shisad defense → verdict) is unusually effective at showing that security is a design property, not a patch set.

**The timeline is damning.** Johann Rehberger reported the underlying vulnerability in 2025 and it was "acknowledged but unresolved." Then wunderwuzzi demonstrated it in October 2025, got the HackerOne report closed as out-of-scope, and had to push to get it re-opened. PromptArmor demonstrated it again in April 2026 against a *new product* (Cowork) that shipped without fixing the root cause. This isn't a zero-day — it's a known-day that Anthropic chose not to address before launching a feature (local folder access) that made it dramatically worse.

**The open questions are the interesting part.** Slow exfiltration over multiple turns is the obvious next attack vector — if you can encode a few bits of data per response across thousands of interactions, no behavioral analyzer catches it. The UX tension around user-intended adversarial files ("I asked you to read this file, why won't you?") is a genuinely hard problem that no one has solved. These questions are more valuable than the attack descriptions themselves.

**The shisad framework may or may not exist.** There's no link to source code, no paper, no repository. It could be lhl's personal project, a company's internal framework, or a thought experiment. The specificity of the defenses (13 YARA rule files, 5-voter consensus, 8-layer PEP pipeline) suggests real implementation experience rather than armchair architecture. But without verification, treat it as a design reference, not a production claim.

**The real lesson for agent builders:** separate the model provider's API from the agent's execution environment. The model API call should happen in a privileged daemon process, not inside the sandbox. If your agent's sandbox can reach your API provider, you have the same vulnerability. This applies to every coding agent — not just Claude.

Related: [[Security and Sandboxing]] for the broader landscape, [[A Deep Dive on Agent Sandboxes]] for Codex's approach, [[yolo-cage]] for sandboxed execution with egress scanning, [[HackAPrompt Dataset]] for injection patterns, [[VTcode]] for OS-native sandboxing, [[claude-ctrl]] for enforcement architecture, [[You Dont Want Long-Lived Keys]] for credential hygiene, [[Benchmark Exploitation]] for emergent exploitation, [[AI Coding Tools Create More Bugs Than They Fix]] for agent safety stats.

---
*Source: [[raw/attack-review-claude-allowlisted-egress-exfiltration]]*
*Last updated: 2026-05-15*
