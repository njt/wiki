# AI Security Framework for DevSecOps

A practical guide to operationalizing AI security across the development lifecycle, from PreEmptive's Michelle Pruitt. The article bridges the gap between high-level governance frameworks (NIST AI RMF, EU AI Act) and the concrete controls DevSecOps teams need: CI/CD enforcement, application hardening, runtime monitoring, and prompt/data security. Its strongest contribution is the "workflow-friendly enforcement" principle — security controls embedded in the build pipeline rather than bolted on after deployment.

---

## Key Quotes

> SAST and SCA remain essential, but they do not fully address AI-specific risks such as prompt injection, insecure output handling, model misuse, or non-deterministic behavior.

This is the article's thesis in one sentence. Traditional AppSec tools operate on code — they can't reason about prompt structure, model behavior at inference time, or the agent chains that emerge when LLMs call tools. The gap isn't that SAST is bad; it's that SAST was designed for a threat model that didn't include language models as runtime components.

> Security controls are more effective when embedded in CI/CD and developer workflows rather than relying on optional manual steps.

The most important sentence in the article and the one most relevant to this wiki's audience. It's the security equivalent of "linters beat prompts" ([[Guardrails and Feedback Loops]]) — deterministic pipeline enforcement beats human diligence every time. If a security check is optional, it won't happen. If it gates the build, it will.

> When you're shipping AI logic in a client application, file, or binary, the security surface extends beyond the backend. Client-side protections matter when AI-enabled logic is exposed.

The argument for application-layer hardening as a distinct security domain alongside network and runtime controls. This is PreEmptive's commercial interest, but the point stands independently: if your app ships with embedded AI logic or model weights, an attacker with a decompiler has unlimited time to study it. [[DeepSeek Reverse Engineers TeamSpeak Licensing]] proved the reverse-engineering barrier collapsed to $3.88 in API credits.

> AI security frameworks are most effective when they are built around practical controls, not just governance.

The operational thesis. Governance documents don't stop prompt injection. CI/CD gates do.

## Key Themes

- **#concept** AI security as a lifecycle-wide concern — the article maps five phases (design, development, build, deployment, ongoing) with distinct controls at each, rejecting the "scan at the end" anti-pattern
- **#pattern** CI/CD-enforced security gates — mandatory, auditable, repeatable checks that block the build or gate the release rather than producing reports nobody reads
- **#concept** Defense-in-depth across the AI stack — models, prompts, APIs, code, dependencies, binaries, runtime behavior all need protection; no single tool covers the whole surface
- **#comparison** Framework taxonomy — NIST AI RMF (voluntary governance), EU AI Act (binding regulation), OWASP Top 10 for LLMs (technical vulns), MITRE ATLAS (threat intel), Google SAIF (implementation principles) are complementary, not interchangeable
- **#tool** Application-layer hardening — obfuscation, anti-tamper, and reverse-engineering defenses for shipped binaries containing AI logic (PreEmptive's commercial angle)

## Critical Analysis

The article does something genuinely useful: it takes the sprawling landscape of AI security frameworks and distills them into a practical checklist for DevSecOps teams. The comparison table (NIST vs. EU AI Act vs. OWASP vs. MITRE vs. SAIF) is the most immediately valuable artifact — a five-minute orientation that saves hours of reading primary sources to understand which reference serves which purpose.

The "workflow-friendly enforcement" principle is the strongest insight and the one this wiki's audience should take seriously. It's the same argument that runs through [[Guardrails and Feedback Loops]] and [[Steering Claude Code]]: if a control requires human memory or willpower, it fails. If it's embedded in the pipeline, it works. The article extends this from code quality to security without saying so explicitly.

The limitations are worth naming. First, the article is a vendor piece from PreEmptive, and the framing reflects it — application/binary protection gets a full section with product placement while equally important domains (prompt injection defense, runtime monitoring tooling) get conceptual treatment without concrete recommendations. The framework comparison table is genuinely useful; the "and PreEmptive covers the application layer" coda is less so.

Second, the article doesn't engage with the hardest AI security problem: prompt injection defense at the model boundary. It mentions prompt injection as a risk and recommends "safe prompt handling," but has nothing concrete to say about how to achieve it. This isn't a failure of the article so much as a reflection of the state of the field — [[Bounding the Blast Radius — Prompt Injection Defenses]] documents that every known defense collapses under adaptive attack — but the article's silence on the difficulty is notable.

Third, the lifecycle framework is strongest on the build-and-release phases where deterministic checks apply (CI/CD gates, binary hardening) and weakest on the runtime phase where AI's non-determinism makes monitoring genuinely hard. "Monitor for anomalous behavior" is good advice but underspecified — what does anomalous mean for a system whose outputs are probabilistic by design?

The healthcare example is thin — it gestures at "secure design, validation, testing, monitoring" without showing what any of those look like for an AI system handling patient data. A worked example with specific controls and failure modes would have made the framework concrete.

Despite these limitations, the article earns its place in this wiki because it's one of the few pieces that connects AI security governance to DevSecOps practice. Most AI security writing sits at the policy level (NIST, EU AI Act) or the vulnerability catalog level (OWASP Top 10). This article lives in the middle — the engineering layer where frameworks become pipeline checks and pipeline checks become shipped software. That middle layer is where most teams actually work, and it's underserved by existing writing.

## Connections

- [[Web Application and API Protection (WAAP)]] — same company (PreEmptive), different security layer. WAAP covers the network perimeter; this article extends into the AI development lifecycle. Together they form PreEmptive's argument for defense-in-depth from network to code to AI
- [[Agentic AI Security Stack]] — Lucktemberg's comprehensive 200+ page reference is the canonical threat model for AI systems; this article is the DevSecOps playbook for operationalizing a subset of those threats
- [[Zero Trust for AI Agents]] — Anthropic's security framework shares the "practical controls over governance" philosophy and the "impossible vs. tedious" test applies equally to the CI/CD enforcement this article advocates
- [[Bounding the Blast Radius — Prompt Injection Defenses]] — the article mentions prompt injection as a risk but doesn't solve it; this page documents why prompt injection defense is genuinely hard and why the article's silence on the difficulty is significant
- [[Guardrails and Feedback Loops]] — "linters beat prompts" is the same structural insight as "CI/CD enforcement beats manual security review"
- [[Cybersecurity Is Proof of Work Now]] — Breunig's compute-economics framing: in an AI-accelerated threat landscape, security is an economic question of whether your defenses cost the attacker more than the attack is worth. The article's application hardening argument depends on this logic
- [[DeepSeek Reverse Engineers TeamSpeak Licensing]] — the most vivid proof that the reverse-engineering barrier has collapsed, making the article's application-protection argument urgent rather than theoretical
- [[Supply Chain Security for Software Developers]] — the article's "model and third-party component risk" section maps directly to the supply chain security practices this page documents

---
*Sources: [[raw/ai-security-framework-devsecops]]*
*Last updated: 2026-07-18*
