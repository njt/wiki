# Project Glasswing — Mythos at Cloudflare

Cloudflare's field report from months of pointing Anthropic's Mythos Preview at over 50 of their own repositories. The piece is unusually substantive for a vendor blog: it documents a nine-stage vulnerability discovery harness, names exactly what Mythos does that other frontier models don't (exploit chain stitching and closed-loop proof generation), and delivers sharp operational lessons about adversarial review, narrow scoping, and why patching faster is a trap. The most important document yet on what production AI vulnerability discovery actually looks like.

---

## Key Quotes

> "A suspected flaw without a working proof is speculation."

This is the line that separates Mythos Preview from everything before it. Other frontier models can find bugs. Mythos writes the PoC, compiles it, runs it, and iterates on failure. The shift is from "here's something that might matter" to "here's something that definitely matters, and here's the code that proves it." That changes triage economics entirely — you go from investigating hunches to deciding whether to fix confirmed exploits.

> The exploit chain reasoning "looks like the work of a senior researcher rather than the output of an automated scanner."

Bourzikas is describing the qualitative leap: not just finding isolated low-severity bugs but reasoning about how to combine them into a single working exploit. Those low-severity bugs "would traditionally sit invisible in a backlog." The scanner finds them but can't connect them; Mythos can.

> Patching faster "does not change the shape of the pipeline that produces the patch."

The most counterintuitive argument in the piece. Many teams now run two-hour CVE-to-patch SLAs. Bourzikas points out that if regression testing takes a day, you're skipping it — and "the bugs you ship when you skip regression testing tend to be worse than the bugs you were trying to patch." Cloudflare learned this by letting the model auto-generate patches that "fixed the original bug while quietly breaking something else." The real answer is architectural: make exploitation harder even when bugs exist.

> Emergent guardrails "aren't consistent enough to serve as a complete safety boundary on their own."

The Mythos Preview version used in Glasswing lacked the additional safeguards in generally available models. It still sometimes refused legitimate security research tasks — but inconsistently. Same prompt, same task, different runs produced different answers. Bourzikas is making a safety argument, not a capability complaint: organic refusals alone can't be trusted as a safety layer.

> Putting "two agents in deliberate disagreement" proved far more effective than telling one agent to be careful.

This is the adversarial review finding and it's one of the most transferable operational lessons. A second agent with a different prompt and model, incapable of generating its own findings, catches noise a self-reviewing agent misses. The same pattern as [[AI Needs to Think Before Giving Feedback]]'s G-E-RG loop and [[Trycycle]]'s fresh-agent review. The mechanism isn't "be more careful" — it's structural disagreement.

---

## Key Themes

#security #AI #harness #multi-agent #vulnerability #Mythos

**Exploit chain construction as the frontier.** The gap between "finds bugs" and "proves those bugs are exploitable" is where Mythos Preview pulls ahead. Other models find the same underlying bugs but stop at description. Mythos closes the loop. This reframes the capability question from "can your model find bugs?" to "can your model deliver a triage-ready exploit?"

**The harness IS the product.** The nine-stage pipeline (Recon → Hunt → Validate → Gapfill → Dedupe → Trace → Feedback → Report) is the real intellectual property. Each stage addresses a specific failure mode of raw model output: wandering attention, hallucinated findings, coverage gaps, duplicate inflation, unreachability. This isn't "point a smart model at a repo." It's a designed system. [[Harness Engineering]] and [[Harness Engineering (OpenAI)]] make the same argument for coding agents; Cloudflare's contribution is demonstrating it for security specifically.

**Adversarial review beats careful prompting.** Stage 3 (Validate) uses an independent agent that can only disprove findings, not generate new ones. This is the same pattern as [[Scaling Long-Running Agents]]'s planner/worker/judge topology and the adversarial review in [[The Lifecycle of a Swamp Issue]]. "Be more careful" is a prompt; "here's an agent whose job is to prove you wrong" is a system.

**Narrow scope as a feature.** "Broad prompts make the model wander; specific scope hints produce researcher-like behavior." This runs directly counter to the instinct to give agents big-picture context. The harness constrains each hunter to one attack class + one scope hint. Coverage comes from parallelism, not per-agent breadth. Same insight as [[Honey I Shrunk the Coding Agent]] — the scaffold shapes behavior more than the model does.

**The patching trap.** Two-hour SLAs create perverse incentives: skip regression testing, ship worse bugs. The architectural alternative — front-end defenses, contained blast radius, simultaneous rollout — is the harder path but the correct one. This is the same argument as [[Security and Sandboxing]]'s emphasis on isolation over patching, applied at Cloudflare's scale.

**Dual use is real and acknowledged.** Bourzikas doesn't dodge: the same capabilities "will, in the wrong hands, accelerate the attack side." Cloudflare's response is architectural — they're building the defensive products for customers at the same time they're building the offensive tooling for themselves. Whether that's a coherent strategy or just a conflict of interest is left as an exercise for the reader.

---

## Critical Analysis

**What makes this important:** This is the first detailed operational account of a frontier security model deployed at scale against production infrastructure. Not a benchmark. Not a lab demo. Cloudflare pointed Mythos at their actual runtime, edge data path, protocol stack, and control plane. The nine-stage harness is the most complete published reference architecture for AI vulnerability discovery. Anyone building in this space will study this post.

**What's genuinely new:** The exploit chain construction capability is the headline. Prior models could find individual bugs. Mythos Preview chains low-severity primitives into working exploits. This isn't an incremental improvement — it changes what kind of work the model can do. The proof-generation loop (write, compile, run, iterate) is the second genuinely new capability. Together they collapse the distance between "suspicion" and "confirmed vulnerability" in a way that changes triage economics.

**What's suspiciously tidy:** The post reads like a joint Cloudflare-Anthropic production. It names Mythos Preview specifically and repeatedly, compares it favorably to unnamed "other frontier models," and frames the shortcomings (emergent refusals) in ways that make the case for Anthropic's safety narrative. That doesn't make it wrong — the operational detail is too specific to be pure marketing — but it's worth reading alongside [[AI Cybersecurity After Mythos — The Jagged Frontier]] for a less vendor-aligned perspective. Fort's finding that 8/8 models (including a 3.6B open-weights model) detect Mythos's flagship exploit complicates the "only Mythos can do this" framing.

**The patching trap is the most important argument and it gets the least space.** Bourzikas buries his most radical claim: that faster patching is actively harmful when it forces you to skip regression testing. This is the architectural counter-argument to [[Cybersecurity Is Proof of Work Now]]'s compute-arms-race framing. Breunig says spend more tokens; Bourzikas says change your architecture so the tokens matter less. Both are right; neither is sufficient alone.

**The open-source skill that seeded this harness** is [[Cloudflare Security Audit Skill]] — the six-phase Claude Code skill (recon → hunt → adversarial validate → report → structured output → independent verify) from which Glasswing evolved. The skill is a single-repo starting point; Glasswing is the fleet-wide, multi-stage production system it became.

**The harness architecture has an obvious missing stage.** The nine-stage pipeline goes Recon → Hunt → Validate → Gapfill → Dedupe → Trace → Feedback → Report. What's missing? A **Remediate** stage. Cloudflare learned the hard way that letting the model write patches produced fixes that "quietly broke something else." The pipeline finds and confirms vulnerabilities but stops at reporting. The gap between "here's the exploit" and "here's the safe fix" remains human-mediated — and that's probably correct given current capability, but it's a gap the post doesn't acknowledge as explicitly as it should.

**The dual-use tension is deeper than the post admits.** Cloudflare runs security products for millions of applications. They're also building offensive AI tooling. The claim that "the architectural principles described mirror those our products deliver for customers" is either a profound business synergy or a conflict of interest that should make customers nervous. The fact that Bourzikas can present it as the former without addressing the latter is a mark of how early we are in this conversation.

**Why this matters for the wiki:** This piece synthesizes nearly every thread in the [[Security and Sandboxing]] synthesis page — harness design, adversarial review, sandboxing, dual-use — into a single operational narrative. It's the best reference implementation document we have for AI security tooling. It also connects the security conversation to the agent architecture conversation in [[Agent Orchestration]] and [[Agent Design & Architecture]] in ways that make both richer.

---

*Sources: [[summary/cyber-frontier-models]]*
*Source URL: https://blog.cloudflare.com/cyber-frontier-models/*
*Author: Grant Bourzikas, Cloudflare*
*Published: 2026-05-18*
*Fetched: 2026-05-18*
