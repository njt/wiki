# Honey, I Shrunk the Coding Agent

Itay Inbar holds the model constant and varies the scaffold, demonstrating that a 9B quantized local model (Qwen3.5-9B) can jump from 19.11% to 45.56% on Aider Polyglot just by redesigning the agent wrapper around it. The custom little-coder scaffold beats Aider's baseline by 2.4x on the same model — and the Aider baseline itself already outperforms several larger models on the public leaderboard. The paper inverts the standard eval framing: benchmark outcomes are properties of model-scaffold interaction, not model weights alone.

---

## Key Quotes

> "Coding agents are built around assumptions of high autonomy, strong long-horizon planning, and reliable tool use" — assumptions suited to frontier models but not to small local models.

The article's thesis in one sentence. Agent frameworks are implicitly tuned to GPT-4-class behavior; drop in a smaller model and the same scaffold becomes mismatched infrastructure.

> "Failures on small models may in fact be evidence of misalignment between the model and the agent wrapped around it."

This reframes small-model benchmark results entirely. The model isn't failing at coding — the scaffold is failing at supporting it.

> "little-coder turns each of them into infrastructure."

Referring to the Write-tool guard, thinking-budget cap, workspace discovery, reference cards, and malformed-output parser. These aren't optional polish — they're the scaffold's operating system. Every failure mode gets a dedicated mechanism.

> "The asymmetry is the scaffold signal."

Of 112 exercises where the two scaffolds disagree, 69 go to little-coder and 11 to Aider. That's a 6:1 ratio. The scaffold isn't nudging performance — it's determining outcomes.

> "benchmark outcomes for coding agents are properties of model weights in interaction with the agent that surrounds them, rather than properties of model weights alone."

The concluding claim. It reads like a truism but the entire eval ecosystem operates as if it were false.

---

## Key Themes

- **#concept** Scaffold-model co-design — The agent harness isn't a fixed container you swap models into; it needs to be adapted to each model's behavioral profile. The write-tool guard fires on 57% of exercises. The thinking budget caps reasoning at 2,048 tokens. These aren't general-purpose optimizations — they're specific responses to specific failure modes of a specific model.

- **#tool** [[little-coder]] — Open-source scaffold built on clawspring substrate. Key mechanisms: Write guard (no new files, Edit only), thinking-budget cap with retry-without-thinking fallback, workspace discovery via conditional keyword injection, tool skill cards (80-150 tokens each) and algorithm cheat sheets capped at ~500 tokens/turn, malformed-output parser and quality monitor.

- **#pattern** Infrastructure-izing failure modes — Instead of prompting harder, build infrastructure for each failure category. The Write guard handles wrong-tool errors. The thinking budget handles reasoning loops. The quality monitor handles empty responses. Each mechanism is cheap (tokens, not training) and each targets a specific, observed failure.

- **#tool** [[Aider]] — Baseline scaffold achieving 19.11% on Polyglot with Qwen3.5-9B, which the author notes is already "the first result I am aware of that places a sub-10B quantized local model at measurable performance on Aider Polyglot" and beats several larger models on the public board.

- **#concept** Time economics of scaffolds — Aider's pass/fail time ratio is ~1.6 (141s vs 224s); little-coder's is ~2.8 (177s vs 492s). Little-coder spends proportionally more time on failures because it tries harder — and it converts enough of those to passes to achieve 4.7 passes/hour vs Aider's 2.75.

---

## Critical Analysis

**The most important benchmark paper nobody will replicate.** Inbar has done something genuinely rare: he held the model constant and varied the scaffold. Almost nobody does this. The entire leaderboard-industrial complex treats the model as the variable and the scaffold as infrastructure — and this paper shows that's wrong by a factor of 2.4x.

**The mechanisms are suspiciously cheap.** No fine-tuning, no LoRA, no RLHF. The adaptations are purely architectural — tool guards, token budgets, reference cards. This means the cost to implement them is near-zero compared to model training. If these results generalize (the author is careful to note they haven't been tested on SWE-bench or real PR workflows), the implication is that scaffold engineering is the highest-leverage activity in coding agents, full stop.

**The gap between claims and ablations is real.** Inbar is honest that "the observables are not ablations" — we don't know which of the five mechanisms actually matters. The write guard fires on 57% of exercises, but maybe it's the thinking budget doing the real work. Formal ablations are the natural next step, but the paper as-is is more existence proof than dissection.

**The Aider baseline is the hidden story.** A sub-10B quantized local model at 19.11% on Polyglot already beats several larger models on the public board. Inbar argues these models were "excluded from the coding-agent evaluation conversation prematurely." He's right. The eval ecosystem has a frontier-model selection bias that systematically undercounts small-model capability.

**The pass/fail time ratio is the metric that matters for production.** Little-coder spends more time on failures (492s vs Aider's 224s) because it's actually trying to recover. That's the right tradeoff for a benchmark. But for a production system where you want fast failure detection, Aider's ratio might actually be preferable. This tension — thoroughness vs fast failure — is underexplored in agent design.

---

*Sources: [[raw/honey-i-shrunk-the-coding-agent]]*
*Last updated: 2026-05-15*
