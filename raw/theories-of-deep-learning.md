---
url: https://astledsa.substack.com/p/theories-of-deep-learning
title: "Theories of Deep Learning"
author: "astle dsa (Theoretical Limits)"
date_fetched: 2026-07-18
date_published: 2026-07-11
---

## Theories of Deep Learning

By astle dsa, July 11, 2026, on Substack (Theoretical Limits)

The author opens by noting that deep learning has seen "exponential empirical success" through architectures and algorithms that worked via scaling, while theory lagged — apart from the Universal Approximation Theorem and its modern extensions. They claim the gap is narrowing, with multiple theories emerging for different aspects of deep learning. This essay offers a high-level overview of theories they've encountered.

**Important caveats from the author:** These are mathematically dense frameworks that either formalize the field or explain observed phenomena. The author states this reflects their personal understanding developed with AI assistance, and they invite corrections. They also note they use "theory" and "framework" interchangeably.

### Section I — Sub-domains of Deep Learning

The author sketches three broad sub-domains before diving into theories:

- **Architecture theory:** Theoretical foundations for model architectures, providing a common language to express transformers, RNNs, CNNs as instances of a general framework.
- **Optimization theory:** Explains and predicts optimizer behavior based on architectures and data distributions, aiming for better optimizers like Adam, AdamW, Muon.
- **Functional Theory:** Focuses on the neural network as a function simulator, explaining generalization and phenomena like grokking/double descent.

The author notes these aren't formal categories — they interconnect — but papers typically address one aspect at a time.

### Section II — The Three Theories

#### 1. Categorical Deep Learning (Architecture Theory)

**Paper:** "Position, Categorical Deep Learning is an Algebraic Theory of All Architectures" (arXiv:2402.15332)

**Authors:** Bruno Gavranović, Paul Lessard, Andrew Dudzik, Tamara von Glehn, João G. M. Araújo, Petar Veličković

The author discovered this paper after exploring Geometric Deep Learning and Topological Deep Learning. While those earlier fields generalize data distributions with internal structures, CDL provides a framework describing **all** model architectures, bridging theory and implementation via category theory concepts grounded in functional programming.

The author quotes the paper (with emphasis added): "lack of a coherent bridge between specifying constraints which models must satisfy and specifying their implementations." The authors propose applying "category theory—precisely, the **universal algebra of monads valued in a 2-category of parametric maps**" as a single theory subsuming both flavors of neural network design.

**Key mechanism:** The authors use monad compositionality to represent sequential computation and recurrence. A 1-category has morphisms between objects (like vector spaces), while a 2-category has morphisms between morphisms, capturing higher-order operations like parameter-sharing, operation variations, and automatic differentiation. The framework is an algebra of monads in a 2-category space.

#### 2. Modular Duality (Optimization Theory)

**Paper:** "Modular Duality in Deep Learning" (arXiv:2410.21265)

**Authors:** Jeremy Bernstein, Laker Newhouse

The author notes deep learning optimization relied on standard algorithms like SGD or Adam that worked in practice but lacked foundational frameworks, forcing reliance on empirical experimentation. This paper proposes a framework illuminating neural network training while deriving faster algorithms.

**Core concept:** A **norm** — a measure of size for a matrix/vector space. Using norms during training ensures updates don't cause unexpectedly large output changes (related to Lipschitz continuity). Different operations (self-attention vs. feed-forward) reside in different normed spaces with unique geometries, so the goal is understanding which norms optimize each layer.

The **Modular norm** composes all norms of a given architecture recursively, enabling a better geometry-aware optimization algorithm. Each network layer is a separate "module" contributing its own norm.

The **Modular Dualization** paper identifies an error researchers had been ignoring: gradients and updates reside in different vector spaces, so subtracting one from the other is mathematically incoherent. The solution: a **Duality Map** transforming update matrices into the correct parameter space. The choice of norm determines the duality map, since weight updates reside in the norm's **dual space**.

**Training procedure described:**
- Construct the modular norm from various layer norms
- Calculate gradient via backpropagation (naturally lives in the norm's dual space)
- Use the duality map to transform the gradient into the desired weight update

The gradient functions as a functional (taking parameter perturbations as input and outputting a real scalar), making it a point in the Dual Space.

#### 3. A Theory of Generalization (Functional Theory)

**Paper:** "A Theory of Generalization in Deep Learning" (arXiv:2605.01172v1)

**Authors:** Elon Litman, Gabe Guo

While classical ML used the bias-variance tradeoff, deeper models exhibited phenomena violating those assumptions: benign overfitting, double descent, implicit bias, and grokking.

The author quotes the key shift: "We propose a radical Vereinfachung[simplification]: abandoning the **parameter space** entirely. Instead, we analyze the network as a dynamical system strictly in **output space**." The focus moves from billions of latent parameters to how predictions evolve and error flows.

**Mathematical tool:** The empirical Neural Tangent Kernel (eNTK). The authors speculate training dynamics based on eNTK eigenvalues, identifying a **signal channel** (higher mobility, easier to learn) and a **reservoir** (lower mobility, harder to learn).

**Explanations offered:**

- **Benign overfitting:** Important features transfer into the signal channel while noise stays in the reservoir. Reservoir contributions are vanishingly small, so overfitting doesn't contaminate test accuracy.
- **Double Descent:** As model size increases, the reservoir enlarges, handling more noise — yielding low test errors even where overfitting would be expected.
- **Implicit Bias:** SGD learns important features faster because the signal channel is more mobile (associated with large eNTK eigenvalues), making those directions favorable. The eNTK couples predictions together, measuring their influence — meaningful shared features align with high-eigenvalue directions (signal channel), while rarer noise is weakened (reservoir).
- **Grokking:** Harder-to-learn features initially reside in the reservoir. Training and parameter updates change output space geometry, potentially shifting these features to the signal channel, where they're learned extremely fast — producing sudden loss decrease.

### Section III — Other Theoretical Efforts

The author notes these three theories each focus on a single aspect, constructing unified frameworks centered on mathematical objects (Prediction space, Output space, Architecture space).

They mention **mechanistic interpretability** as a related sub-field, citing Anthropic's research on induction heads, circuits, attribution graphs, and the J-space (global workspace theory).

Additional references (not deeply explored by the author) include scattered research papers not presented as overarching theories, with links provided to three alphaxiv papers. Book-length papers establishing deeper mathematical foundations are also cited (two alphaxiv links and arXiv:2106.10165).

The author notes the CDL lead author extended their framework to cover every aspect of deep learning in a thesis (arXiv:2403.13001), which the author hadn't fully reviewed.

### Section IV — Conclusion

The author chose these three theories for their stronger foundational frameworks. Future posts will explore implementations and dive deeper into theorems and proofs.

### References Cited

1. Universal Approximation Theorem and modern extensions (arXiv:2407.12895)
2. Geometric Deep Learning resources (two links)
3. Topological Deep Learning (arXiv:2206.00606)
4. Categorical Deep Learning paper (arXiv:2402.15332)
5. Modular Duality paper (arXiv:2410.21265)
6. Theory of Generalization paper (arXiv:2605.01172v1)
7. empirical NTK paper (arXiv:2206.12543)
8. Norm mathematics (Wikipedia)
9. Lipschitz continuity (Wikipedia)
10. Metrized Deep Learning (modula.systems)
11. Dual space (Wikipedia)
12. Elon Litman's blog post on theory of deep learning
13. Anthropic research on induction heads, circuits/attribution graphs, and global workspace
14. Additional alphaxiv references: 2512.19199, 2606.05599, 2605.18598
15. Book-length papers: arXiv:2407.18384, arXiv:2603.18387, arXiv:2106.10165
16. Extended CDL thesis: arXiv:2403.13001
