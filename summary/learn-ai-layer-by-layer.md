---
url: https://learnai.robennals.org/
title: Learn AI Layer by Layer
author: Rob Ennals
date_fetched: 2026-05-31
date_published: unknown (work in progress; first release covered through Transformers)
---

# Learn AI Layer by Layer

An interactive guide to understanding modern AI from first principles. Built by Rob Ennals for his 11-year-old son Isaac, after finding existing AI tutorials either too vague, required too much expertise, or weren't engaging enough.

Source code: https://github.com/robennals/ai-explained
Substack: https://messyprogress.substack.com

## Structure

The tutorial is divided into 27 planned chapters plus appendixes. As of the first release, chapters 1–9 (through Transformers) and appendixes A1 (PyTorch from Scratch) and A2 (Glossary) are published. Chapters 10–27 are marked "Coming soon."

Each chapter includes interactive browser-based playgrounds where readers can manipulate sliders and see concepts in action, plus an optional Google Colab notebook for readers who want to work with real PyTorch code at a deeper level.

Target audience: anyone with "a middle-school level understanding of math."

## Chapter Summaries

### Introduction
The project began because Ennals wanted his son to understand modern AI. Existing tutorials fell short — too vague, too presumptuous about expertise, or too dry. His goal: build something understandable by anyone with middle-school math that still provides deep intuitive understanding of how and why AI works. Each chapter uses interactive playgrounds — "playing with an idea makes it easier to grasp than just reading about it." Colab notebooks for advanced readers. The transformer was chosen as the milestone for first release.

### Chapter 1: Everything Is Numbers (Computation)
Three foundational claims: (1) everything inside an AI is numbers — text, images, sound; (2) an AI is a function — numbers in, numbers out; (3) the function itself is made of numbers — billions of parameters called "knobs." A model is a machine with adjustable knobs — different settings produce different functions. The Church-Turing thesis: a machine can compute any computable function, so if thinking is computation, AI is possible in principle.

The real challenge: brute force via lookup table is impossible — a tiny 8×8 grayscale image has 256⁶⁴ possible inputs (a number with over 150 digits, dwarfing the atoms in the observable universe). What's needed: models that compress — capturing patterns with far fewer parameters than entries — plus automated parameter finding, which we call "learning."

### Chapter 2: The Power of Incremental Improvement (Optimization)
(Fetched but content limited by restrictions. Core concept from navigation: gradient descent — small improvements add up to find good solutions step by step. The same algorithm behind biological evolution, A/B testing, and the scientific method.)

### Chapter 3: Building a Brain (Neural Networks)
A neuron does two things: weighted sum of inputs (plus bias), then activation function (sigmoid). The sigmoid is an S-curve — same shape as adoption curves, population growth: exponential start, linear middle, diminishing returns. A single neuron is a "smooth logic gate": it can compute AND, OR, NOT with the right weights, but changes smoothly when weights adjust — which makes it trainable. XOR requires two layers: hidden layer computes OR and NAND, output layer ANDs them.

Minsky and Papert's 1969 proof that single-layer networks can't solve XOR "killed most neural network research for over a decade." The fix was using more than one layer. Hidden layers build stepping stones: raw inputs → simple patterns (edges) → parts (eyes, ears) → whole objects (cat, dog). Depth adds abstraction at each layer. GPT-3 had 96 layers; frontier models have more. Training uses backpropagation — "flows the error signal backward through each layer" — so a network with ten thousand weights is no harder to train "in principle" than one with ten.

Size comparison: today's largest models have roughly as many parameters as a mouse brain has synapses, yet can write essays while mice cannot. "The number of connections isn't everything" — weight quality matters more.

### Chapter 4: Describing the World with Numbers (Vectors)
Vectors are lists of numbers describing things. The dot product measures similarity — multiply matching pairs, sum the results. Large positive = similar, near zero = unrelated, negative = opposite. Unit vectors (magnitude 1) constrain dot products to [-1, +1], sometimes called cosine similarity.

Any vector decomposes into a unit vector (what kind of thing) and magnitude (how much of that thing). This decomposition is "one of the most important ideas in the chapter."

The big reveal: a neuron's weighted sum IS a dot product. The weight vector's unit vector defines *what* the neuron detects; its magnitude sets sensitivity; the bias sets baseline excitement. Magnitude and bias do related but different jobs: "The magnitude sets how steeply the curve rises while the bias slides the curve sideways."

### Chapter 5: From Words to Meanings (Embeddings)
Modern AI rejects universal dimensions — "How big is the color blue? How fast is democracy?" don't make sense. Instead: put similar words close together and let directions mean different things in different places. One number line can encode both category (which region) and attribute (position within region). Multiple dimensions allow overlapping categories and fine-grained distinctions.

This is an **embedding**: a learned mapping from discrete symbols to continuous geometry. The dimensions don't need labels — "what matters is that similar things are nearby and that directions encode meaningful relationships."

In a well-trained embedding (like GloVe: 400K words, 300 dimensions), directions carry meaning. The direction from "tiny" to "huge" captures size; "rabbit" to "elephant" captures the same concept through mammals. Vector arithmetic: "elephant − mouse" = "bigger animal" vector; add to "rabbit" → lion, giraffe, rhino. "Paris − France" = "capital of." Each transformation is "extracted by subtraction and applied by addition."

But structure is "real, but it's shaped by what the model was trained on, not by some idealized geometry of meaning." Directions only exist where training gave the model a reason to organize words along them.

Subword tokenization: real models break text into tokens (whole words, word parts, punctuation). The embedding layer is the first neural network layer — one-hot encoding activates one input per token, weights become the embedding. GPT-2: 768 dimensions. GPT-3: 12,288 dimensions.

### Chapter 6: Understanding by Predicting (Next-Word Prediction)
The chapter's thesis: "Prediction REQUIRES understanding." To predict "The capital of France is ___" you need geographic knowledge; "two plus two equals ___" requires arithmetic.

Bigram model: count word co-occurrences from a corpus, predict most frequent follower. Controlled by **temperature** — low picks the most likely word, high distributes evenly. The n-gram wall: with a 50K vocabulary, 50,000¹⁰ possible 10-word sequences (47-digit number). "Most 10-word sequences the model encounters will be ones it has never seen before." Need to generalize.

Solution: neural network fed embeddings of prior words. "Cat" and "dog" have similar vectors — learning "the cat ___" → "was" automatically generalizes to "the dog ___" → "was." Fixed window size is still limiting — meaning often depends on text far earlier. A novel establishes someone as a disgraced surgeon; thousands of words later, "John picked up the scalpel" requires that context.

On the "just predicts the next word" dismissal: prediction demands broad world knowledge, and modern LLMs don't only predict next words — that's how they learn, but many other techniques teach them to apply that knowledge productively. The Friston free energy principle: the brain itself may be a prediction machine. "If Friston is right, then learning by prediction isn't a cheap shortcut to intelligence" — it may be fundamental.

### Chapter 7: Paying Attention (Attention)
"The breakthrough that makes modern AI work." Words need other words for context — "bank" disambiguation, pronoun resolution. Attention lets the model "look at every word that came before and decide which ones matter."

Query/Key/Value architecture: each token gets three vectors. Query = what it's looking for; Key = what it offers; Value = what gets communicated if selected. Match scores via dot product of query and key. Softmax converts to percentages summing to 100% — "a competition: bigger numbers dominate, smaller numbers get squeezed."

The sink problem: softmax always allocates 100% somewhere, even when no real match exists. Solutions: a sink token with a slightly nounish key to absorb leftover attention; a "none" dimension so off-task tokens search in the none dimension and only the sink answers. Real models don't always use dedicated sinks — punctuation or start-of-sentence tokens sometimes serve the purpose.

Multi-headed attention: each head has its own Q, K, V, "each free to learn a different pattern." Each head specializes — pronouns, grammar, structure, semantics. Multiple heads "let the model ask many different questions in parallel."

The model learns "a simple one-layer neural network (without an activation function) that transforms the token's current embedding into query, key, and value vectors." Each head gets its own three weight sets — "how different heads end up looking for different things."

### Chapter 8: Where Am I? (Positional Encoding)
Attention is position-blind: "The dog bit the man" and "The man bit the dog" produce identical scores. Two solutions presented:

**ALiBi** (Attention with Linear Biases): subtract a distance penalty from attention scores — `score = (query · key) − m × distance`. Different heads use different slopes. Simple and effective but limited: each head uses one fixed slope for all queries, and the falloff is always linear.

**RoPE** (Rotary Position Embeddings): uses a property of dot products — the dot product of two unit vectors depends only on the angle between them. Rotate each token's query/key by position × speed. Nearby tokens = small angle difference = high dot product. Different dimension pairs rotate at different speeds — fast pairs capture local structure, slow pairs span the entire context. Each token specifies an arbitrary distance penalty curve by choosing weights across speed bands.

RoPE doesn't double dimensions — it pairs adjacent dimensions as (x, y) coordinates. Rotation preserves length (√(x²+y²)) while changing direction. Causal masking ensures tokens only attend backward (rotation encodes distance, not direction).

Nearly every frontier model uses RoPE: Llama, Mistral, Gemma, Qwen, DeepSeek.

### Chapter 9: One Architecture to Rule Them All (Transformers)
"The architecture behind every major AI you've heard of: ChatGPT, Claude, Gemini, Llama." Text passes through a sequence of layers; each layer improves each token's representation by blending what's known about that token with information from other tokens. Attention determines which tokens carry relevant information.

The key difference from regular neural networks: "in a normal neural network the wiring is fixed, learned once during training." Attention dynamically determines where information flows. Each node also contains its own small neural network merging incoming information with existing state.

GPT-3: 96 layers with 96 heads per layer, 12,288-dimensional embeddings. "Latest models are believed to be significantly larger." Modern LLMs have "enough capacity to encode complex meanings like a blue object belonging to a human astronaut, observed in the Martian sky from a planet in Earth's solar system."

Interpretability is an open problem: "we usually don't actually know what the intermediate representations of tokens mean." The transformer isn't optimizing for human-understandable layers — it's "simply seeking representations that best help it achieve its objective."

Each transformer layer (or "transformer block"): attention locates relevant tokens, then a feed-forward network merges that information with existing knowledge. Stack enough of these and meaning emerges from prediction.

### Appendix A1: PyTorch from Scratch
A hands-on introduction to PyTorch. Two core capabilities: fast tensor math (GPU-accelerated) and automatic differentiation (computes gradients through any computation). Google Colab recommended — no local install needed, free GPU, pre-installed PyTorch.

Covers: tensors (containers of numbers), shape, basic math, matrix multiplication, autograd (`requires_grad=True`, `.backward()`), `nn.Linear` (weighted sum with bias), `nn.Sequential` (layer stacking), activation functions, the training loop (forward → loss → zero grad → backward → update). Complete XOR example: 2→8→1, Adam optimizer, 2000 epochs.

Key point: "Without a nonlinearity in the middle, two stacked linear layers collapse mathematically into a single linear layer."

### Planned Chapters (10–27, Coming Soon)
Thinking by Rotating (matrix math), Why Training Almost Doesn't Work, Mixture of Experts, Long Context, Running Models Fast (inference/hardware), Looking Inside the Mind (interpretability), Learning from Experience (RL), Self-Play, Reasoning Models, Alignment, Distillation and Synthetic Data, Image Comprehension, Image Generation, World Models, Audio, Agents and Tool Use, Hallucination and Grounding, Context Management.

## Technical Approach

Interactive playgrounds use browser-based widgets (adjustable sliders, real-time computation). The attention chapter lets users click individual components in a transformer block to see what each does. The embeddings chapter provides a live word-similarity explorer against a subset of GloVe. Vector arithmetic is demonstrated with real embedded words.

Each chapter includes a Colab notebook implementing the concepts in PyTorch: building neurons, training XOR networks, one-hot encoding, word embeddings with analogies, bigram models with temperature sampling, attention from scratch with heatmap visualization, ALiBi, RoPE, full transformer training.

## Design Philosophy

- Start with concrete interactive demos, not notation
- Build each concept from the previous one — the tutorial is strictly sequential
- Reveal the math as a description of what the widgets already taught
- Target the middle-school math floor but don't pull punches on depth
- Each chapter ends with a single quiz question (multiple choice, one right answer)
- "Playing with an idea makes it easier to grasp than just reading about it"
