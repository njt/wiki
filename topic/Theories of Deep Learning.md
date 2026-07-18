# Theories of Deep Learning

A survey of three emerging mathematical frameworks — Categorical Deep Learning, Modular Duality, and output-space generalization theory — that together represent the most serious attempt yet to give deep learning the theoretical foundation it has lacked since the 2012 ImageNet moment.

---

## The Theory Gap

Deep learning has been a field where practice runs ahead of understanding. Architectures and optimizers work because they work, not because anyone can prove why. The Universal Approximation Theorem told us networks *could* approximate anything but said nothing about how they actually learn, why they generalize, or what architectures are optimal. Three recent papers are closing that gap from different angles.

## Categorical Deep Learning: An Algebra of Architectures

> "lack of a coherent bridge between specifying constraints which models must satisfy and specifying their implementations" — Gavranović et al.

This is the most ambitious claim in the essay: a single algebraic framework that describes *all* neural network architectures. The tool is category theory — specifically, monads in a 2-category of parametric maps. If that sounds abstract, it is. But the payoff is concrete: transformers, RNNs, and CNNs become instances of the same algebraic structure rather than ad-hoc designs. The 2-category formalism captures parameter-sharing, operation variations, and automatic differentiation as higher-order morphisms.

The CDL lead author has since extended this into a thesis (arXiv:2403.13001) claiming to cover every aspect of deep learning. The ambition is totalizing — and almost certainly overclaims — but the core insight that architecture design is algebra struggling to be born feels right. The field has been designing architectures by intuition and benchmark-chasing; CDL offers the prospect of deriving them from principles.

## Modular Duality: Fixing the Math of Optimization

> Gradients and updates reside in different vector spaces, so subtracting one from the other is mathematically incoherent.

This is the essay's most startling claim: deep learning practitioners have been doing optimization math wrong. The gradient of a loss with respect to parameters lives in the *dual space* of the parameter space — it's a linear functional, not a vector. Subtracting it from parameters (which live in the primal space) is a category error. The fix is a Duality Map that depends on the choice of norm for each layer.

The practical upshot: each layer type (self-attention, feed-forward, convolution) operates in a different normed space with its own geometry. The Modular norm composes these recursively, and the resulting optimization algorithm is geometry-aware in a way that SGD and Adam are not. This is #optimization theory that could actually produce better optimizers — Muon already gestures in this direction.

**The take:** This paper diagnoses a real mathematical sloppiness that everyone ignored because the optimizers worked anyway. Whether the correction produces meaningfully better training in practice remains to be seen, but the diagnosis is sharp.

## Generalization Without Parameters

> "We propose a radical Vereinfachung: abandoning the parameter space entirely."

Litman and Guo's move is the most conceptually elegant of the three. Instead of reasoning about billions of parameters, treat the network as a dynamical system in output space and analyze the empirical Neural Tangent Kernel. The eNTK's eigenvalue spectrum naturally partitions into a **signal channel** (high mobility, learns fast) and a **reservoir** (low mobility, learns slow).

From this single mechanism, they derive explanations for four phenomena that classical ML theory couldn't handle:

- **Benign overfitting:** signal goes to the fast channel, noise stays in the slow reservoir where it can't hurt test accuracy
- **Double descent:** larger models have larger reservoirs, absorbing more noise without contamination
- **Implicit bias:** SGD favors high-eigenvalue directions, which correspond to shared, meaningful features
- **Grokking:** features stuck in the reservoir can be promoted to the signal channel by training dynamics, producing sudden phase transitions

This is #theory doing what theory should: taking disparate empirical surprises and showing they're consequences of one mechanism. The output-space move is genuinely novel — reminiscent of how thermodynamics sidesteps particle-level detail by working at the macro scale.

## What's Missing

The essay is honest about its limitations. These three theories each address one sub-domain — architecture, optimization, generalization — and there's no unified framework connecting them. The CDL thesis claims to bridge this gap, but the author admits they haven't fully reviewed it. Mechanistic interpretability ([[Jacobian Lens]], [[Global Workspace in Language Models]]) operates at yet another level of analysis, and its relationship to these mathematical theories is unclear.

More fundamentally, none of these theories yet makes *falsifiable predictions* that could guide architecture or optimizer design. They explain what we've already observed but haven't yet told us something surprising that turns out to be true. That's the acid test for a scientific theory, and it's still ahead.

The essay is also explicitly an AI-assisted survey by a non-specialist — the author flags this up front. It's a map of the territory, not original research. But as a map, it's unusually clear about what each theory claims, what mathematical machinery it uses, and how it relates to the others.

## Why This Matters

Deep learning's theoretical poverty has real costs. Without theory, architecture design is alchemy, optimization is folklore, and we can't predict when models will fail. These frameworks — especially the output-space generalization theory and the modular duality diagnosis — suggest we're entering a phase where deep learning becomes a proper science rather than a enormously successful craft.

The essay also captures a specific moment: mid-2026, when multiple theoretical threads are converging and the gap between practice and understanding is visibly narrowing. If even one of these frameworks produces a novel, verified prediction in the next year, it will be the most important development in deep learning since the transformer.

---

*Sources: [[raw/theories-of-deep-learning]], [[summary/theories-of-deep-learning]]*
*Last updated: 2026-07-18*
