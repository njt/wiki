# Playing with Vision Embeddings

Preston Jensen reverse-engineers DINOv3's 384-dimensional vision embedding space using sparse autoencoders, feature visualization, and direct manipulation — showing what a small vision transformer actually "sees" and how its features compose. A masterclass in making neural network internals legible without language models.

---

## Key Quotes

> "Embeddings are, in a sense, the native language of neural networks. They are how networks can encode a rich variety of semantically meaningful representations with just a list of numbers. However, those numbers are frustratingly opaque."

The thesis in two sentences. Jensen frames the problem not as "how do we build better embeddings" but "how do we read the ones we already have." This is mechanistic interpretability's core question, applied to vision rather than language. The whole post is a demo of what you can do when you refuse to treat embeddings as black boxes.

> "DINOv3 encodes far more than 384 distinct visual concepts into those 384 numbers. How? The leading hypothesis is something called superposition: models learn to cram many times more features than the dimensionality of their embeddings by pointing each feature in a nearly-orthogonal direction."

The superposition demo — squeezing 10 MNIST classes through a 2D bottleneck — is the post's best pedagogical move. Most superposition explanations stay in math; Jensen shows you the optimizer fighting to arrange 10 classes in 2 dimensions in real time. In 384 dimensions there's room for thousands. This is why a 12,000-feature SAE isn't overkill — it's necessary to unsmear the compressed representation.

> "When you add corn kernels with a triumphal arch, you get a fusion of the two — an arch made of corn kernels. But when you add corn kernels with screws, you get the two features juxtaposed with each other — screws on top of corn."

The feature arithmetic section is where the post gets genuinely surprising. Adding feature vectors is supposed to be linear: combine "corn" and "arch," get "corn-arch." But the model doesn't treat all feature pairs the same way. Some compose by fusion, others by juxtaposition. Jensen doesn't solve why, but the observation itself is the contribution — it means SAE features don't live in a simple vector space where addition means "blend." The geometry is weirder than that.

> "Feature 1511 is a feature specifically for single, large, whole strawberries, and 2314 is about many small strawberries whether whole or sliced."

The strawberry deep-dive is the post's most rigorous section. Through systematic experiments varying size, count, and cut-state, Jensen isolates exactly what distinguishes two seemingly-similar features. This is feature splitting (Chanin et al., 2024) made concrete: what looks like redundant encoding is actually a granular distinction the model learned independently. The key insight: "we only looked at two in depth here, but our SAE generated twelve thousand. Interpreting all of them is incredibly difficult, and doing a manual analysis is not scalable." The post is honest about the ceiling on manual interpretability.

---

## Key Themes

- **Superposition as the fundamental challenge of interpretability** — Models pack more features than dimensions by using near-orthogonal directions. This makes single dimensions uninterpretable and requires tools like SAEs to disentangle. #concept
- **Sparse autoencoders as the microscope** — 32× expansion (384 → 12,288), L1 sparsity penalty, dead feature resampling. The SAE gives you 12,000 interpretable directions from 384 opaque ones. #tool #pattern
- **Feature visualization through optimization** — Differentiable models let you invert the embedding: start from noise, optimize pixels to maximize cosine similarity with a target direction. The augmentation trick prevents high-frequency cheating. #pattern
- **Feature arithmetic isn't linear in practice** — Some feature pairs compose by fusion (corn + arch = corn-arch), others by juxtaposition (corn + screws = screws beside corn). The geometry of SAE feature space has structure we don't yet understand. #concept
- **Feature splitting means every "obvious" category is actually many** — Two strawberry features that look similar turn out to encode size, count, and wholeness independently. 12,000 features from a 384-dim embedding means massive splitting throughout. #concept
- **Manual interpretability doesn't scale** — The post demonstrates depth (two strawberries analyzed exhaustively) while acknowledging breadth is impossible by hand. The field needs automated interpretability tools. #pattern

---

## Critical Analysis

This is one of the best pieces of ML explainer writing I've read. Jensen has the rare combination of technical depth and visual design skill — the interactive sliders and feature carousels aren't garnish, they're the argument. You can't convey "superposition" with text alone; you need to see the 10-class-2D bottleneck animation. You can't understand feature arithmetic without the side-by-side generated images. The post's format *is* its methodology.

The choice of DINOv3 over CLIP or SigLIP is smart. Language-aligned vision models (CLIP, SigLIP) are more practically useful but harder to interpret because language bleeds into the embedding structure — you're never sure if a feature is "visual strawberry-ness" or "textual association with the word strawberry." DINOv3's purity (no language, no text supervision) makes the interpretability claims cleaner. What you're reading is genuinely vision, not vision-through-language.

The honesty about the generation pipeline's artifacts is important and easily missed. The generated images are more saturated, higher contrast, and duplicate objects. Jensen tells you this upfront so you can mentally correct for it. Most feature visualization work buries these caveats in appendices.

The biggest limitation — which Jensen acknowledges — is that this is a single small model (ViT-S, 384-dim CLS embedding). The techniques almost certainly generalize to larger vision transformers, but nobody's run this playbook on a ViT-L or a vision-language model yet. The open question at the end ("Can you do similar feature visualization in Vision Language Models?") is the right one. VLMs have richer embeddings but the language contamination problem makes interpretability much messier.

The UMAP map at the end is pretty but underdelivers relative to the rest of the post. It shows feature clusters exist, which we already knew from the decomposition examples. What it doesn't show is whether those clusters correspond to semantic categories humans would name — the interactive version might, but the static frame in the post doesn't.

---

## Related Pages

- [[Learn AI Layer by Layer]] — The best free AI foundations tutorial; covers embeddings and attention with interactive widgets. Jensen's post is what that pedagogy looks like applied to cutting-edge research
- [[A Non-Anthropomorphized View of LLMs]] — Halvar Flake's argument that neural networks are functions through ℝⁿ, not proto-minds. Jensen's feature-by-feature dissection is the empirical version of this claim
- [[KV Cache Locality]] — Another "here's what's actually happening inside the model" post, applied to inference infrastructure rather than interpretability
- [[Where the Goblins Came From]] — A miniature interpretability crisis in production: what happens when you don't understand your model's internal features. Jensen's tools are what you'd use to prevent this
- [[Recent Developments in LLM Architectures]] — Context for where vision transformer research sits relative to the broader model landscape
- [[2025 in LLMs]] — Simon Willison's landscape survey; useful context for where mechanistic interpretability fits in the field

---

*Sources: [[summary/playing-with-vision-embeddings]]*
*Last updated: 2026-06-09*
