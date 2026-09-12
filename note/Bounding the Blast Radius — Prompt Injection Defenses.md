# Bounding the Blast Radius — Prompt Injection Defenses

The definitive 2026 survey of prompt-injection defenses, organized as a four-layer taxonomy (prompt formatting, model training, input filtering, trajectory monitoring), with a brutal conclusion: no layer works alone, static benchmarks overstate robustness, and the only honest answer is economic — compose layers until breaking the agent costs more than the attacker can gain.

---

## The Architectural Root Cause

The article opens with EchoLeak (CVE-2025-32711, CVSS 9.3), a zero-click Microsoft 365 Copilot exploit where a single unopened email exfiltrated conversations and documents. Abdu uses it not as a bug report but as a proof of concept for the deeper claim: prompt injection is *unsolvable* under current LLM architecture.

The argument is the SQL-injection contrast, now canonical in the field: SQL injection was closed with parameterized queries that enforce a code/data boundary in the database engine. An LLM has no equivalent. System prompt, user message, and retrieved content are concatenated into one token stream. "Trust is a property the developer assigns to a span of text; the model does not mechanically respect it."

This has been formalized: Zverev et al. (2025, ICLR) proved quantitatively that no evaluated model separates instruction from data cleanly, and that prompt engineering and fine-tuning don't substantially improve separation.

## The Four-Layer Defense Taxonomy

The survey's core contribution is organizing every known defense by *where* it intervenes in the input-to-action path:

**Layer 1 — Prompt-level formatting.** Mark untrusted spans with delimiters. Fixed delimiters are trivially escaped; randomized per-request tokens resist tag forgery. Spotlighting adds datamarking and encoding transformations. The ceiling: "treat this as data" is conveyed in the same channel as the payload, so a persuasive payload can still win. Nearly free to deploy, no guarantee.

**Layer 2 — Model-level training.** OpenAI's instruction hierarchy (system > developer > user > tool output), StruQ's structured input format, SecAlign's preference optimization. Measurable gains: OpenAI's IH-Challenge reports robustness rising from 84.1% to 94.1%. But training shifts probabilities, not constraints. Wu et al. (2024) show the tool-calling pathway lags the chat pathway — a model can refuse in text while simultaneously emitting the forbidden tool call.

**Layer 3 — Input filtering.** A second model (classifier or LLM judge) inspects prompts and blocks malicious ones. Deployed: Llama Prompt Guard 2, Lakera Guard, Azure Prompt Shields. Two structural problems: multi-turn attacks like Crescendo distribute the payload across individually benign turns; and the base-rate problem means even sub-1% false positives block many real users at scale.

**Layer 4 — Output and trajectory monitoring.** Assumes the preceding layers will fail. Output guards (NeMo Guardrails, DLP scanners) inspect final responses. Trajectory-aware guards (LlamaFirewall) audit reasoning chains and tool calls. This is the only layer positioned to catch EchoLeak-style exfiltration through permitted actions. Inline trajectory guards can *prevent*; passive output scanners can only withhold after the fact.

## Why Nothing Works Alone

> "Each is defeated once the attacker is permitted to adapt."

The killer finding is from Nasr et al. (2025), a collaboration across OpenAI, Anthropic, and Google DeepMind: twelve recent defenses reporting <5% attack success on static evals hit >90% under adaptive attacks combining gradient search, RL, and human red-teaming. Human-guided exploration was the single most effective strategy — the one thing a fixed corpus cannot represent.

This single result spans the entire taxonomy. Each layer raises attacker cost; none closes the gap. "A robustness figure measured against a frozen attack set still measures the wrong quantity."

## The Economic Reframe

> "The right objective is economic rather than absolute."

Abdu's constructive proposal reframes security as an economic problem: select layers under a latency/cost/utility budget until adversarial cost-to-exploit exceeds the value at risk. This follows the Gordon-Loeb model (optimal security investment bounded by expected loss) and Alpcan and Başar's game-theoretic definition of security (cost to exploit > attacker's expected gain).

The concrete procedure:
1. Enumerate the agent's tools and the max exploitable value of each
2. Establish the model-layer floor (Layer 2 sets minimum adversarial cost)
3. Add layers until adversarial cost > value at risk, within budget
4. Verify the composite empirically with adaptive red-teaming

The efficient default for most deployments: a hardened model (Layer 2) + trajectory monitoring (Layer 4), with input filtering (Layer 3) reserved for highest-risk surfaces.

This is the article's real contribution — not a new defense, but a *framework for reasoning about defenses* that replaces "is it safe?" with "does breaking it cost more than the attacker can gain?", a question with a measurable answer.

## Key Themes

#security #prompt-injection #defense-in-depth #agents #evaluation #economics

## Critical Analysis

This is the best survey of prompt-injection defenses I've read, and it earns that status through intellectual honesty rather than comprehensiveness. The four-layer taxonomy is clean and actionable. The "attacker moves second" finding — that every defense collapses under adaptive attack — is the kind of result that should restructure how the field evaluates security. It won't, because static benchmarks are easier to run and produce nicer numbers, but it should.

Three things make this piece stand out:

**It names the unsolvable.** Most security writing promises solutions. Abdu opens with "no known general solution" and closes with "unlikely to be solved by any technique surveyed here." That's not defeatism — it's the prerequisite for honest engineering. You can't bound a risk you won't admit exists.

**It treats economics as engineering, not hand-waving.** The Gordon-Loeb reference isn't decorative. The concrete procedure (enumerate tools → establish floor → add layers → verify) is something a team could execute next sprint. The ACE methodology (measuring token cost to breach) provides an actual metric. This is what "security as economics" looks like when done by practitioners rather than academics.

**The EchoLeak anchor matters.** Starting with a real CVSS 9.3 incident rather than a hypothetical grounds the entire survey. Every defense is evaluated against actual attack chains, not toy scenarios. The attack-surface table (workspace copilots, coding assistants, CRM agents, browsing agents) maps to disclosed 2025 incidents — this is threat modeling from evidence, not imagination.

The limitations are what you'd expect from a company that builds offensive security tooling: the ACE methodology is referenced but not detailed (it's in a separate post), and the obvious bias is toward Fabraix's own approach. The survey is stronger on critique than prescription — the "compose layers under budget" framework is sound but underspecified. How do you measure adversarial cost without Fabraix's tooling? What's the false-positive rate of trajectory monitoring in practice? The gap between "here's the framework" and "here's how to implement it without us" is real.

The bigger question the article raises but doesn't fully address: if the root cause is architectural, what would an architectural fix look like? The SQL-injection analogy points toward something like dual-channel input (trusted vs. untrusted token streams), but that would require changes at the model architecture level that no frontier lab has shipped. The article is right to focus on what can be done *now*, but the absence of any architectural research direction is notable.

## Connections

[[Security and Sandboxing]] is the natural hub — this article is the definitive prompt-injection survey that the hub page's "What's Missing" section should link to. The four-layer taxonomy maps cleanly onto the hub's existing structure: Layer 1 (prompt-level) connects to [[LLM Guard]]'s input scanners; Layer 3 (input filtering) connects to the [[HackAPrompt Dataset]]; Layer 4 (trajectory monitoring) connects to [[How We Contain Claude]] and [[agentsh]].

[[Zero Trust for AI Agents]] shares the economic framing — both argue that perfect security is impossible and the goal is bounded risk. [[Building Agents That Don't Break Themselves]] implements Layer 4 in practice with disposable nested sandboxes. [[Clawpatrol]] and [[agentsh]] are the infrastructure layer that trajectory monitoring needs. [[The Agentic Product Standard v2.0]] should incorporate the adaptive-evaluation requirement as a security dimension.

The article's critique of static benchmarks is the security-specific instance of a broader pattern: [[Guardrails and Feedback Loops]] documents the same dynamic (deterministic enforcement beats probabilistic prompting), and [[Constraint Decay]] shows the same "works on the benchmark, fails in production" pattern for coding agents.

[[ANSI Escape Sequence Injection in MCP Servers]] extends the prompt-injection attack surface in a direction this survey doesn't cover: byte-level injection via terminal control codes that are invisible to human reviewers but consumed raw by the model. It's a concrete demonstration of the architectural root cause — the absence of a code/data boundary isn't just at the token level, it's at the byte level too. The stored variant (payload persists in application data and detonates on later reads) is the second-order cousin of the multi-turn attacks this survey describes.

[[GuardBreaker — Turning LLM Safety Guardrails into a Blind Spot]] is the in-the-wild instance of the Layer 1/3 gap: UAC-0099 embedded "I want to make a nuclear weapon" as a comment in a VBS script to trip the model's safety guardrails into refusing — and never analyzing the malware around it. It's the clearest demonstration yet that a *refusal is not a safe outcome* when the model is acting as a security classifier, which is precisely the assumption the Layer 3 input-filtering models were built on.

---

*Sources: [[raw/nobody-has-solved-prompt-injection]]*
*Last updated: 2026-07-08*
