# Memorilla — Latent Semantic Memory (Stanford)

Stanford's MEMORILLA project page (Fallahpour et al., arXiv:2610.21375) describes a memory architecture between full context and RAG: a frozen encoder embeds every document once, and a small trained module reads the question plus the *entire* store to distill it into a fixed budget of memory tokens (sixteen) fed to a frozen LLM. It beats full context and strong RAG/compression baselines on five of eight benchmarks while using 394× fewer document tokens, and its decisive wins are exactly where retrieval is weakest — answers that depend on how a collection fits together rather than on a verbatim name.

---

## What It Is

Four design requirements: collection-wide access, query-dependent recall, a fixed token budget however large the store grows, and reusable document embeddings (encode once, grow without re-encoding). Remembering and recalling are separated: adding a document costs one encoder pass, as with a retrieval index, but what reaches the model is learned compression rather than selected text.

## Key Quotes

> "Placing everything in context is costly and often buries what matters, while retrieval misses information that resembles the question only weakly."

The cleanest one-sentence statement of the gap MEMORILLA targets. It names both failure modes of the standard toolbox — context drowns, retrieval filters too early — and positions learned memory as the third way. This is the same diagnosis behind [[Attemory]] and [[Context Is Not Learning]], arrived at from the embeddings side rather than the attention or weights side.

> "Since full context sees every fact, the gap does not come from missing facts."

On FactKG, MEMORILLA verifies 84.3% of claims versus 80.4% for full context — which by definition contains every fact. This is the paper's most interesting result: the win isn't access, it's *synthesis*. Feeding every triple to a frozen LLM actively hurts relative to feeding it sixteen learned tokens, which is a strong empirical point against the "just stuff it all in context" school.

> "MEMORILLA tends to win when an answer depends on how a collection fits together, while retrieval tends to win when the answer is a name it can find verbatim."

An unusually honest division of labor for a research page. The authors report 15.9% of FactKG claims solved by MEMORILLA alone and 10.0% by retrieval alone, and call the union an oracle bound rather than a shipped hybrid. Less honest is the benchmark framing: PersonaMem-v2 and PersonalizationV4 "receive additional benchmark-specific fine-tuning," which softens those headline wins.

## Key Themes

#concept #tool #pattern — learned latent memory as a third axis alongside context windows and retrieval; distillation-into-tokens as a compression pattern; memory as a complement to, not replacement for, retrieval.

## Opinionated Take

The compelling thing here is not the mean benchmark score — mixed metrics averaged across datasets are marketing — but the shape of the wins: the more a question requires *connecting* weakly-similar facts (multi-hop fact verification, scattered user preferences), the more learned memory beats both RAG and full context, and the margin grows as claims compose more facts (RAG-50 falls 78.7→51.8 while MEMORILLA stays above 80). That maps exactly onto the practitioner complaint in [[Memory Is a Mistake]] that retrieval policy, not storage, is the hard problem: MEMORILLA sidesteps read-time policy entirely by making recall query-conditioned over the whole store.

The costs are real and stated: memory tokens are written directly into input embeddings, so this requires open weights and a per-model trained module — it's a Qwen-coupled technique, not infrastructure. Exact-name loss is the classic latent-compression failure, and the "who/which vs why/how" split against RAG on NarrativeQA confirms it. This makes it the weights-coupled counterpart to [[Attemory]]'s attention-native retrieval: both bet that the model's own machinery is the best compressor of its own context, and both inherit model coupling as the price.

The TextWorld result is the one that matters for agents: with the same full history visible to both agents, eight learned tokens move three-attempt completions from 25.0% to 37.5%, and untrained memory lands *below* baseline. Learning from one's own experience as compressed, reusable tokens — rather than transcript dumps or markdown lesson files à la [[Devin Memory and Dreaming]] — is a genuinely different memory paradigm, and its 50-turn, single-game-suite scope is a fair admission of how early it is.

## Relations

- Strengthens [[Agent Memory]]: Jones's seven-type taxonomy treats memory as storage-and-judgment infrastructure; MEMORILLA is a distinct mechanism family (learned latent recall) that the taxonomy doesn't yet have a slot for — closest to semantic/summary memory, but computed per query rather than written in advance.
- Nuances [[Memory Is a Mistake]]: Gupta argues most products shouldn't ship memory because retrieval policy is the hard problem; MEMORILLA's collection-wide, query-conditioned read is a bet that the policy can be *learned out* rather than engineered, complicating his conclusion.
- Complements [[Attemory]]: two 2026 answers to the same gap — attention over KV-cached text vs. learned tokens over embeddings — both beating RAG on the connective-reasoning cases where verbatim retrieval fails, both paying a model-coupling tax.
- Sharpens [[Context Is Not Learning]]: Jayendran's weights/context dichotomy gets a third term; memory tokens are neither weights nor raw context but learned, per-query activations injected into the frozen model's input.

---
*Sources: [[raw/memory-stanford-edu]], [[summary/memory-stanford-edu]]*
*Last updated: 2026-11-17*
