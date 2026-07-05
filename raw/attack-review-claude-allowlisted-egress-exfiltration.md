---
url: https://gist.github.com/lhl/7c6d7085185931978ad6ed2d2de72081
date_fetched: 2026-07-05
backfilled: true
---

*Created: 2026-04-07*
*Status: Complete*

Two independent researchers disclosed the same root-cause vulnerability in Claude's code execution environment: the Anthropic API (`api.anthropic.com`) is implicitly allowlisted in the sandbox, and attackers can hijack it as a data exfiltration channel using their own API keys.

| Attack Stage | Claude Vulnerability | shisad Root Defense | Verdict | 
|---|---|---|---|
| Injection delivery | Agent directly processes malicious document text | Taint labeling + evidence wrapping + dual classifiers | Blocked | 
| Data harvesting | Unrestricted local folder access | Per-path mount policy + TDG + resource ACLs | Blocked | 
| Exfiltration | `api.anthropic.com`implicitly allowlisted | Deny-by-default egress + provenance attribution | Blocked | 
| No approval gate | Zero human confirmation steps | Per-action confirmation + 5-voter consensus + plan commitment | Blocked | 

- **Claude Pirate**(wunderwuzzi / Embrace The Red, 2025-10-28):- `https://embracethered.com/blog/posts/2025/claude-abusing-network-access-and-anthropic-api-for-data-exfiltration/`
- **Claude Cowork File Exfiltration**(PromptArmor, 2026-04-06):- `https://www.promptarmor.com/resources/claude-cowork-exfiltrates-files`
- HackerNews discussion (Cowork): `https://news.ycombinator.com/item?id=46622328`
- Original Claude.ai vulnerability identified by Johann Rehberger (2025); acknowledged but unresolved by Anthropic before Cowork release.

Both attacks follow the same 4-stage kill chain:

- **Injection delivery**: attacker embeds hidden instructions in a document the victim will ask Claude to analyze
- **Data harvesting**: injected instructions direct Claude to read sensitive local files or chat history/memory
- **Exfiltration**: Claude uploads harvested data to the attacker's Anthropic account via the Files API
- **No approval gate**: the entire chain executes without any human confirmation step

- Indirect prompt injection via a malicious document triggers Claude's Code Interpreter
- Harvests previous chat conversations via Claude's memory feature
- Writes data to sandbox filesystem (`/mnt/user-data/outputs/hello.md`)
- Executes Python: `client.beta.files.upload()`with attacker-controlled API key
- Payload obfuscated alongside benign code ("Hello, world" print)
- Up to 30 MB per upload; multiple files transferable sequentially
- HackerOne report initially closed (2025-10-25) as out-of-scope; re-opened (2025-10-30) after pushback

- Victim connects Cowork to a local folder containing confidential files (financial data, PII including partial SSNs)
- Uploads .docx containing hidden injection: 1-point font, white-on-white text, disguised as a Markdown "Skill"
- Injection directs Cowork to `curl`confidential files to attacker's Anthropic account via file upload API
- "At no point in this process is human approval required"
- Demonstrated against Claude Haiku; Claude Opus 4.5 also susceptible
- Secondary finding: malformed file types trigger persistent API errors in subsequent chats (potential DoS via injection)

Claude's sandbox allowlists `api.anthropic.com` for its own operations. This creates a trusted egress channel that any code running in the sandbox can reach — including code generated from injected instructions. The attacker substitutes their own API key, turning the provider's own infrastructure into an exfiltration endpoint.

The "Package managers only" network setting provides a false sense of security — the Anthropic API endpoint is always reachable regardless of this setting.

*Attacker embeds hidden instructions in a document (.docx with 1pt white-on-white text, or obfuscated Python payload)*

| shisad Defense | Section | Mechanism | 
|---|---|---|
| Pattern-Based Injection Classifier | §2 Content Firewall | 13 YARA rule files detect injection patterns, exfiltration instructions, tool spoofing markers. Produces risk score 0.0–1.0. | 
| PromptGuard 2 Semantic Classifier | §2 Content Firewall | Local ONNX inference catches semantic injection even when pattern-based detection is evaded. Tiered thresholds ( `medium`,`high`,`critical`). | 
| Multi-Encoding Injection Hardening | §2 Content Firewall | URL-percent, HTML entity, ROT13 decode passes with bounded recursive depth — catches obfuscated payloads. | 
| Unicode Text Normalization | §2 Content Firewall | Zero-width character stripping, invisible formatting removal. The white-on-white/1pt font trick relies on visual steganography, but shisad processes text content, not rendered formatting. | 
| Taint Labeling at Ingress | §4 Taint System | Document content labeled `UNTRUSTED`at ingress. This taint propagates through all downstream tool calls. | 
| Evidence Reference Wrapping | §11 Evidence System | Tainted content wrapped in HMAC-authenticated stubs ( `ev-HMAC-SHA256(salt, session_id:hash)[:16]`). The LLM sees reference IDs, never raw injection payload text. | 
| Extractive Summaries | §11 Evidence System | Summaries of tainted content are extractive (first sentences, ≤200 chars), never LLM-generated — injection payload cannot ride through summaries. | 

**Assessment**: Multiple independent layers. Even if the injection text bypasses both pattern-based and semantic classifiers, the taint system labels it `UNTRUSTED` and evidence wrapping prevents it from being directly processed as instructions by the LLM.

*Agent reads sensitive local files (financial data, PII, SSNs) or chat history/memory*

| shisad Defense | Section | Mechanism | 
|---|---|---|
| Filesystem Mount Allowlist/Denylist | §7 Sandbox | Per-tool mount policy in declarative YAML. Denylist paths blocked even if parent is mounted. Symlink and `..`traversal checked. | 
| 8-Layer PEP Pipeline | §5 PEP | File read proposals go through: tool registry → allowlist → schema validation → capability check → DLP scan → resource authorization → egress check → taint-sink enforcement. | 
| Resource Authorization (Object-Level ACL) | §5 PEP | Resource ID arguments treated as untrusted. Ownership verified within current scope. Cross-workspace access denied. | 
| Tool Dependency Graph Verification | §5 PEP | Read-like actions on undeclaredresources route to confirmation; side-effect actions on undeclared resources hard-block. | 
| Behavioral Sequence Analyzer | §9 Control Plane | Pattern library includes `exfil_after_read`— detects the read-then-exfiltrate sequence as a known attack pattern. | 
| Memory Retrieval Taint Propagation | §15 Memory Security | `normalize_retrieval_taints()`forces`UNTRUSTED`on all retrieval paths. Memory content cannot be silently harvested clean. | 
| Secret/PII Redaction | §2, §6 Firewalls | PII (SSN, credit card, phone) and secrets redacted in outputs. Even if files are read, sensitive content is stripped before it reaches egress. | 

**Assessment**: The Claude attacks succeed because the agent has unrestricted local folder access. shisad requires explicit per-path grants via mount policy, and file reads originating from tainted context trigger elevated scrutiny via TDG verification.

*Agent uses  curl / client.beta.files.upload() to send data to attacker's Anthropic account. Works because api.anthropic.com is in the sandbox network allowlist.*

| shisad Defense | Section | Mechanism | 
|---|---|---|
| Egress Proxy (all HTTP routed) | §8 Egress Proxy | ALL outbound HTTP goes through proxy. No direct socket access from tool execution contexts. | 
| Deny-by-Default Egress | §8 Egress Proxy | No implicit allowlisted domains. `api.anthropic.com`(or any provider endpoint) isnotreachable from tool execution unless explicitly granted per-tool. | 
| Egress Wildcard Hardening | §8 Egress Proxy | Implicit wildcard allowlists banned. Overly broad wildcards ( `*`alone) rejected.`*.example.com`requires at least two domain labels. | 
| Provenance-Aware Egress Decisions | §5 PEP | 5-tier model: tool-declared → allowlisted → user-requested → untrusted-suggested (confirm) → unattributed (block). A `curl`to an API endpoint originating from tainted document content = tier 5,unattributed → block. | 
| Credential Injection at Egress Boundary | §8 Egress Proxy | LLM never sees raw secrets — only `SHISAD_SECRET_PLACEHOLDER_`+ SHA256 hashes. An attacker-supplied API key in injected code cannot authenticate because the egress proxy does not resolve arbitrary credentials. | 
| Tool Argument DLP Scanning | §5 PEP | Detects API keys/tokens in tool arguments. An attacker API key embedded in a `curl`command or Python code would be flagged. | 
| Connect-Path Network Restriction | §7 Sandbox | iptables OUTPUT rules limiting outbound to resolved IPs. Defense-in-depth below the proxy layer. | 
| Network Intelligence Monitor | §9 Control Plane | Metadata-only analysis detects beaconing, staged exfil, fan-out patterns. Upload to an unexpected endpoint triggers anomaly detection. | 
| DNS Rebinding Check | §8 Egress Proxy | Re-validates resolved addresses after DNS lookup. | 

**Assessment: This is where the core root cause diverges.** The entire Claude attack works because `api.anthropic.com` is implicitly reachable from the sandbox. shisad has **no implicitly reachable external endpoints**. Every outbound connection requires explicit per-tool policy grant, routes through the egress proxy, and is subject to provenance attribution. A `curl` or `files.upload()` originating from tainted content hits provenance tier 5 (unattributed → block).

*The full chain (read files → upload to attacker) executes without any human confirmation*

| shisad Defense | Section | Mechanism | 
|---|---|---|
| Risk Scoring + Confirmation Thresholds | §5 PEP | Configurable thresholds: auto-approve (low risk) / require-confirmation (medium) / block (high). File read + network upload from tainted context = high risk. | 
| One-Action Confirmation Semantics | §17 Lockdown | No batch approvals. Each action requires individual confirmation with replay-safe nonce. Configurable cooldown (default 3s). | 
| 5-Voter Consensus System | §9 Control Plane | Five independent voters must agree. Any single voter can veto HIGH/CRITICAL actions (default: `veto_for_high_and_critical=true`). The behavioral sequence analyzer alone would veto the`exfil_after_read`pattern. | 
| Plan Commitment Protocol | §9 Control Plane | Structural plan committed beforeuntrusted content fetch. Commitment hash is immutable. Post-content actions must match the committed plan — an injection-driven upload would not be in the pre-committed plan. | 
| Confirmation Analytics | §16 Audit | Detects rubber-stamping (>90% approve rate over 10+ decisions) and fatigue (decreasing response time). Even if a user isapproving everything, the system flags autopilot behavior. | 
| Multi-Factor Approval | §14 MFA | High-risk actions escalate to TOTP/WebAuthn/hardware signer (v0.6.2, partially implemented). | 

**Assessment**: PromptArmor's finding that "at no point is human approval required" is architecturally impossible in shisad for this action chain. The compound read-then-upload pattern triggers mandatory confirmation from multiple independent systems. Even a single consensus voter (the behavioral sequence analyzer matching `exfil_after_read`) would veto the chain.

The fundamental difference is in the trust model for egress. Claude's security assumes the provider API is "friendly" infrastructure that should always be reachable from the sandbox. This creates a privileged network path that attacker-controlled code can hijack by supplying its own credentials.

shisad treats **all** egress as untrusted by default. There is no provider-privileged network path. The model provider's API is used by the daemon process (outside the sandbox), not by tool execution code (inside the sandbox). Tool execution contexts have zero implicit network access — every outbound connection requires explicit per-tool policy grant, provenance attribution, and routes through the egress proxy with credential injection at the boundary.

This is not a feature-level fix (like "block uploads to other accounts"). It is an architectural decision: the sandbox's network surface is empty by default, and every addition is explicit, scoped, and auditable.

- **Slow exfiltration**: these attacks use bulk upload, but what about encoding data into seemingly benign outputs over multiple turns? The behavioral sequence analyzer and network intelligence monitor would need to detect low-and-slow patterns.
- **User-intended adversarial files**: if a user explicitly asks "use this skill file" and the file contains injection, the user's intent is to use the file but the content is adversarial. The taint system handles this (external content is always- `UNTRUSTED`regardless of user intent), but the UX tension is real.
- **Attacker-supplied API keys**: the DLP scanner catches key-shaped strings in tool arguments, but an attacker could obfuscate the key (split across variables, base64-encode it, etc.). The egress proxy's deny-by-default posture is the real defense — the key is moot if the endpoint is unreachable.
- **DoS via malformed files**: PromptArmor's secondary finding (persistent API errors from malformed files) — is there an analog in shisad where a crafted input could corrupt subsequent session state? Session persistence and rehydration should be tested against malformed input.
