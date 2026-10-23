# Poisoning Attacks on LLMs Require a Near-constant Number of Poison Samples

An arXiv paper (2510.07192, Oct 2025) showing empirically that poisoning attacks against LLM pretraining need only ~250 poisoned documents no matter how large the model or dataset — a near-constant attack budget that upends the percentage-based threat model most defences implicitly assume.

---

The pre-existing picture of training-data poisoning assumed the adversary controls a *percentage* of the corpus, so defenders could reason about dilution: a big enough clean dataset swamps a fixed quantity of poison. This paper runs the largest pretraining poisoning experiments to date — models from 600M to 13B parameters, chinchilla-optimal datasets from 6B to 260B tokens — and finds the number of poisons required is essentially flat. 250 documents compromised models across all sizes, including the largest, which had trained on more than 20× more clean data. Ablations over poisoned-to-clean ratios and non-random poison placement, plus fine-tuning-stage experiments, confirm the same dynamics.

---

## Key quotes

> "This work demonstrates for the first time that poisoning attacks instead require a near-constant number of documents regardless of dataset size."

The central inversion. Percentage-based reasoning about poisoning is structurally miscalibrated: the attack surface scales with *documents touched*, not corpus share. Any defence argument of the form "our corpus is too big for poison to matter" is now empirically bankrupt.

> "250 poisoned documents similarly compromise models across all model and dataset sizes, despite the largest models training on more than 20 times more clean data."

A concrete, memorable number. 250 is small enough to be operationally realistic for an adversary — no bulk data-acquisition campaign needed. The 20× clean-data multiplier bought nothing.

> "Our results suggest that injecting backdoors through data poisoning may be easier for large models than previously believed."

Note the direction of surprise: scale, usually invoked as a safety virtue, here *worsens* the threat by holding it constant while increasing what a backdoored model can do. The authors' own conclusion is a call for more defence research, not a mitigation recipe.

---

## Analysis

What makes this paper strong is that it is a scale experiment, not a theory paper: they actually pretrained at 13B/260B-token scale and still got flat attack budgets. The main caveat is that "250 documents compromise the model" is about the specific backdoor trigger setups tested — the transfer of the result to every attack class and to frontier-scale runs beyond 13B is inference, not measurement. It's also worth noting the attack model requires *getting poison into the training set*, which for a closed frontier lab is hard; the easiest real-world surface remains open-web-scale data collection and fine-tuning pipelines, where the paper's fine-tuning results are directly relevant.

The uncomfortable synthesis: as models increasingly train on agent-collected, user-contributed, and synthetic data, the number of injection points grows even as per-point vetting stays impossible at 260B tokens. For the wiki's security thread, this paper shifts the poisoning threat model from "how much of the corpus does the adversary own" to "can they get a few hundred documents in at all" — a much lower bar.

## Themes

#concept #security #research

## Related

- [[Agentic AI Security Stack]] — treats supply-chain and data-integrity threats to agents as one layer of a stack; this paper supplies the empirical numbers that make the training-data layer's threat constant-scale rather than proportional.
- [[GuardBreaker — Turning LLM Safety Guardrails into a Blind Spot]] — a different route to the same worry: attacks aimed at the model's trusted internals, showing safety mechanisms as attack surface rather than shield.
- [[Ship Safe]] — the practitioner-facing security checklist for LLM applications; this paper adds a training-time threat that post-hoc application defences cannot see.
- [[Supply Chain Security for Software Developers]] — frames dependency trust for human developers; this paper is the model-weights analogue, where the "dependency" is the training corpus.

---
*Sources: [[raw/2510-07192]], [[summary/2510-07192]]*
*Last updated: 2026-10-24*
