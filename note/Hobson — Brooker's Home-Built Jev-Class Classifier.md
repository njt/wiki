# Hobson — Brooker's Home-Built Jev-Class Classifier

Marc Brooker — AWS principal engineer, distributed-systems eminence, author of several sources already in this wiki — spends two weeks teaching himself AI-model-building by replicating the System One model class at home: "Hobson", a ~2B-parameter calibrated classifier on a single RTX 3090 that lands top of its size class on the public JevBench leaderboard. It is both a technical field report (pointer heads, self-distillation, temperature calibration, 100ms p50 latency) and a rare first-person account of an expert learning a new field *through coding agents*.

---

## What He Built

- **Architecture**: Qwen3.5-2B torso with the LM head removed — no text generation at all — replaced by a ~1M-parameter **pointer head** that scores each option by dotting the hidden state at the `<answer>` position against each option's hidden state. Fine-tuned with a rank-16 LoRA. His first attempt, a 53k-parameter "slot head" with 24 fixed output slots, failed on positional bias and an option-count ceiling.
- **Training**: 115k rows (113k public, 2k synthetic hard examples), one epoch, cross-entropy with shuffled options, plus **self-distillation** — KL to a frozen torso and to earlier checkpoints — to stop catastrophic forgetting.
- **Calibration**: per-question-type temperatures fit on held-out data to minimize log loss, applied at inference by scaling the logits.
- **Results**: top of his size range on jevbench public; Brier 0.009 and 100% accuracy on the easy set; p50 ~100ms, p95 <300ms on a 3090; one or two forward passes regardless of question count (O(1) in questions, thanks to caching the torso pass).
- **Honest limits**: generalization to unseen task types is "useful, but isn't great"; in-task accuracy moved far more easily than generalization; synthetic hard examples helped less than he hoped ("there's a good amount of juice left").

---

## Key Quotes

> "It all comes down to how sub subconsciously intellectually honest I am. Science is hard."

The best sentence in the post. Brooker has seen the JevBench examples and designed the synthesis process, so his hold-out is contaminated in a way no metric can detect — an unusually candid statement of the benchmark-gaming problem that plagues every leaderboard, written by someone usually on the systems side of it.

> "Every line of code was written by an agent, but at each step I tried to make sure the core ideas and insights were mine, or at least I understood them."

The agent-learning pattern: agents as a custom textbook that quizzes you, not as a replacement for understanding. He explicitly scopes this — "I might not set such a bar for a project at work" — which is a sharper claim than most agent-optimism essays: for *learning* projects, comprehension is the deliverable; for production, it's negotiable.

> "People who had a bit of a, ah, *problem* with World of Warcraft or Diablo II should probably find another way to spend their time."

The leaderboard as slot machine. Coming from the person who *built* a leaderboard entry, this is a warning about the "number go up" reward loop that benchmark culture runs on — and a caveat on his own results being optimized for exactly that loop.

> "As I've evolved the model, in-task accuracy has been much easier to move than generalization."

The quiet technical core: at 2B, the model does the tasks it was trained toward, and only grudgingly transfers. He suspects a bigger torso would fix it — the rules of his game don't allow that, which is precisely what makes the experiment informative.

---

## Themes

#concept — calibrated decision models as a buildable commodity, not a lab product
#pattern — head-swapping (LM head → pointer head) as the cheap route to a new model class
#tool — consumer hardware (3090) as sufficient for frontier-adjacent work at small scale
#person — Marc Brooker, systems engineer crossing into model-building

## Analysis

This is the most valuable kind of Jev-adjacent source in the wiki: not the vendor's pitch, not a critic's takedown, but an independent replication with the flaws left in. The results substantially **strengthen** the System One thesis — a solo engineer with one GPU reproduces the class's headline properties (calibration, low latency, parallel questions), which means the moat is not the architecture. But it also **complicates** it: the vendor's calibrated-confidence story rests on exactly the kind of self-graded hold-out Brooker admits is unverifiable, and his "generalization is useful but not great" is a datum the launch announcement never volunteered. His failed slot head is also a useful negative result — the *shape* of the head matters for positional bias, which matters for anyone building decision models on top of torsos.

The meta-narrative deserves equal weight. A principal engineer with three years in AI infrastructure still needed two weeks of agent-tutored study to build this; the agents wrote all the code, but the *ideas* had to be his, and he reports needing several more passes before the algebra sticks. That is a concrete data point for the comprehension-debt debate: agents can compress the lookup, not the understanding — and the person who tried to keep the understanding reports how hard that is even when he's trying.

## Related Pages

- [[System One Models and Jev]] — this source is the independent, from-scratch replication of the model class that announcement defines; it strengthens Jev's core thesis (calibration + latency is buildable) while complicating its marketing (the hold-out integrity problem applies to the incumbent too).
- [[Jev Can't Be Calibrated]] — that critique argued calibration claims were distribution-dependent and hard to verify; Brooker's post is a constructive middle case — calibration *can* be engineered (temperature fitting, Brier measurement) but only against a hold-out whose honesty is a human virtue, not a property of the model.
- [[Local Qwen Is Not a Worse Opus]] — the same "local small model as a different tool, not a worse one" conclusion, here taken further: not just *running* Qwen locally but training on it, with the torso-as-commodity implication that open-weight bases are the substrate for a whole class of purpose-built models.
- [[Building with Jev Skill]] — Drew Breunig's skill for programming against Jev assumes a hosted decision API; Brooker shows the same primitive can be owned end-to-end at home, which reframes "confidence-gated act/confirm/hand-off routing" as something you can self-host.

---
*Sources: [[raw/engineering-system-one-html]], [[summary/engineering-system-one-html]]*
*Last updated: 2026-10-03*
