# Latent Programming Horizons in Coding Agents

Silva, Tu, and Monperrus (KTH, 2026) open a new front in coding agent research: mechanistic interpretability. Using linear probes on residual-stream hidden states collected across 22,714 agentic trajectories, they show that coding agents internally encode program properties (correctness, parse status, regressions) at AUC up to 0.83 — and, more strikingly, that these representations run ahead of the agent's own edits by ~25 steps. They call this the "latent programming horizon."

---

## Key Findings

**Agents know more than they say.** Linear probes — logistic regression classifiers trained on hidden states — decode whether the current code parses, passes tests, reduces failures, or introduces regressions. Full Correctness hits AUC 0.83 on SWE-Bench-Verified with Qwen3.6-35B-A3B. This isn't near-perfect decoding, but it's well above chance, and it holds across two models (Qwen, Laguna XS.2) and two benchmarks (SWE-Bench-Verified, SWE-Bench-Pro).

> "We show that linear probes can decode program properties from the residual streams of coding agents, achieving an AUC of up to 0.83 for full correctness."

The critical detail: these probes use *hidden states*, not output tokens. The agent's internal representation of the program is richer and more structured than what surfaces in its tool calls or natural language commentary.

**The programming horizon is ~25 steps.** The paper's most provocative finding: train a probe to predict program correctness *k steps in the future*, and it performs above chance for at least 25 steps, with signal still detectable at k=50. The agent isn't just reacting — it's implicitly planning, maintaining a running model of where the code is headed.

> "Probes predict future edit outcomes above chance up to roughly 25 steps in advance, which we term the agent's latent programming horizon."

This is the first evidence of anything resembling a planning horizon inside a coding agent. Not explicit planning — these are linear probes reading out directional tendencies from hidden states — but a form of implicit anticipation that survives the noise of multi-step tool-use trajectories.

**The inverted-U across layers.** Probe accuracy follows a consistent inverted-U: weakest at layer 1, peaks in intermediate layers, drops slightly at the final layer. This matches the pattern seen in other probing work — early layers process surface features, middle layers build semantic representations, and the final layer compresses toward token prediction, losing some of the internal structure.

**Transfer is cheap.** Probes trained on SWE-Bench-Verified transfer to SWE-Bench-Pro with only a 0.04–0.09 AUC drop. The representations aren't benchmark-specific; they capture something general about program state.

## Critical Analysis

**This is correlation, not causation — and the paper is honest about it.** The authors explicitly flag that decodability ≠ causal use. Just because a linear probe can read correctness from hidden states doesn't mean the agent *uses* those representations to make decisions. This is the standard caveat in probing work, from Alain & Bengio onward, but it bites harder here because the natural follow-up — "can we steer the agent by intervening on these representations?" — is wildly tempting and completely unproven.

**The 25-step horizon might be an artifact of trajectory structure.** SWE-bench tasks have median 2 edits across 52 steps. Many steps are read-only (file inspection, test execution, error reading). A probe that predicts "will the code be correct 25 steps from now" might be detecting that the agent is *still investigating* versus *starting to edit* — a coarser signal than genuine program planning. The paper's horizon analysis excludes the final k_max steps, but the distinction between "the agent plans ahead" and "the agent hasn't started the hard part yet" deserves more scrutiny.

**Well-formedness is a near-trivial property that the probe mostly can't decode.** On SWE-Bench-Verified, 92%+ of code parses correctly, and the probe collapses to near-chance performance. This is actually a *good* sign — it means the probes aren't just detecting surface regularities that any shallow heuristic could catch. But it also means one of the four properties is basically a null result dressed up as a negative control.

**Two models, both open-weight, both 2048-dim residual streams.** The homogeneity of the model set is a real limitation. Would these results hold for proprietary models with different architectures? For much smaller models? For much larger ones? The paper can't say, and the transfer result (Qwen encodes ~0.10 AUC stronger than Laguna) hints that model scale and training matter.

**The real contribution is the experimental protocol, not the numbers.** Collecting 22.4M hidden-state vectors from 22,714 trajectories across two benchmarks is a serious engineering effort. The paper establishes a template that future work can apply to new models, new scaffolds, and new properties. The 0.83 AUC will age; the protocol won't.

## Why This Matters

This paper sits at the intersection of two trends that have been developing in parallel: mechanistic interpretability of language models ([[Jacobian Lens]], [[Global Workspace in Language Models]]) and the engineering of coding agents ([[Components of a Coding Agent]], [[Constraint Decay]]). Until now, these conversations haven't really touched. Silva et al. wire them together.

The practical implication is speculative but significant: if coding agents maintain structured internal representations of program state, those representations could become a *steering surface*. Instead of prompt engineering or scaffold redesign, you could intervene directly in latent space — nudge the agent toward correct solutions, away from regressions, or toward stopping when it's about to break things. The authors gesture at this: "monitoring and steering coding agents from within the latent space."

The philosophical implication is that [[A Non-Anthropomorphized View of LLMs]] — models as functions through ℝⁿ — gets a concrete demonstration. These representations aren't mystical; they're geometrically structured directions in a 2048-dimensional space that a linear classifier can read. The geometry is real, even if we don't yet know whether the agent *uses* it.

The connection to [[LLM-as-a-Verifier]] is also worth noting: if internal representations encode correctness, they could provide dense reward signals for RL post-training, potentially with better sample efficiency than sparse outcome-based rewards.

**The bottom line:** This isn't a paper that changes how you build coding agents tomorrow. It's a paper that changes what questions you ask about them. The latent programming horizon is a new concept, and like most new concepts in science, its value is in the experiments it enables, not the number attached to it.

---

*Sources: [[raw/latent-programming-horizons-coding-agents]]*
*Last updated: 2026-07-18*
