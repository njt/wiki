# AstaBrief 8B (Ai2)

Ai2's open-weights 8B model (Qwen3-8B + SFT/DPO) that turns a research question and retrieved excerpts into a fully cited scientific report in one pass — now Asta's Fast mode at 51s per report vs 178s for the Claude pipeline, with citation quality competitive against much larger proprietary systems.

---

## What it is

AstaBrief is the model behind "Generate a report" in [Asta](https://allenai.org/blog/astabrief), Ai2's agentic platform for scientific work. Unlike a chatbot answer, a report here is a working research artifact: users bring substantial context and constraints (method, population, setting), and return to the reports later. Because it's open weights, institutions can run it behind their own firewall — necessary when research questions touch sensitive or unpublished work.

## The training recipe: data over optimization

The team explicitly considered an RL path (their own DR Tulu showed RL improves long-form report generation) and chose against it:

> RL-based training can be unstable and expensive. We wanted to see how far we could push report generation quality with a cheaper, more operationally manageable setup—one that's also easier to debug and iterate on.

The bet that follows — spend the effort on data quality rather than fancier optimization — is the most transferable claim in the piece. SFT used 47K full-report targets generated from 90K filtered real user queries by a mix of Claude 3.5/3.7 Sonnet, o3, o4-mini, and GPT-4.1. DPO pairs (~6K) pitted the ScholarQA pipeline against alternative generators (o3, o4-mini, DeepSeek-V3/R1), with GPT-4.1 and DeepSeek-R1 as dual judges at 95% human agreement; only unanimous pairs survived. Multiple generators plus required judge agreement means no single model's output or judgment is treated as ground truth.

## The filtering lesson

Four statistical filters were tested on synthetic training examples; the winner was the simplest:

> The strongest gains came from filtering out synthetic reports with low citation density; more aggressive filtering, filter combinations, and learning-rate sweeps didn't add meaningful gains.

> Scientific specialization, in other words, isn't necessarily a matter of adding more scientific text to pretraining; the composition and quality of post-training data and whether it demonstrates behaviors like grounding and attribution can materially change how the resulting model performs.

That second claim is quietly important: domain specialization arrived through *post-training data that demonstrates the behaviour you want*, not through domain-heavy pretraining.

## The scope problem they name but don't solve

> A model can cite the right study and still make a stronger claim than the study itself supports. This can happen in subtle ways, for example, turning a finding about a particular sample into a generic claim about an entire population, shifting a result reported in the past tense into a present-tense statement that sounds more universally true, or turning a descriptive finding into a recommendation for what clinicians, policymakers, or researchers should do.

Each step "broadens the apparent scope of the evidence without introducing an obviously false statement" — so citation precision/recall alone can't catch it. They flag this as the next evaluation frontier: measuring whether a model preserves the evidentiary *scope and strength* of its sources, not just whether a citation exists.

## Key themes

- #concept — Data quality over optimization method: SFT+DPO with rigorous filtering beat RL-based training, and citation density was the highest-leverage single filter.
- #concept — Evidentiary scope preservation as the missing eval: correct citations can still overstate claims.
- #tool — Open weights as institutional infrastructure: firewall deployment for sensitive research questions.
- #pattern — One-pass generation: training the model to emit the whole report at once removed the expensive summarise/cluster stages with no quality loss.

## Analysis

The genuinely opinionated takeaway is that Ai2 used a frontier model to teach a small open model to *behave differently*, not to know more. The behaviours — cite every claim, don't drift, write in one pass — are exactly the kind of thing RL was supposed to buy, and getting them via careful SFT/DPO data construction is a cheap, reproducible alternative worth stealing for any domain-specific fine-tune.

The honesty about evaluation limits is rare and welcome: most "our model matches GPT-4 on X" posts would never volunteer that their metrics can't detect subtle scope inflation. But it also means the strongest claim is narrower than it looks — AstaBrief matched a *2025* Claude pipeline, and they say so. The adoption numbers (23% of Fast-mode triers never went back; 18% mode-switch with Fast at ~40% of threads) are the more convincing evidence that quality held.

One thing to watch: the "write the full report in one pass" trick trades the pipeline's intermediate scaffolding (summarisation, clustering) for end-to-end learning. That works while the domain is narrow and the retrieval is good; whether it survives messier multi-tool scientific workflows is exactly the future work they list (multi-turn, multi-tool, query decomposition).

## Relations

- Strengthens [[Local Deep Research]]: both are cited-report generators that run locally, but LDR bets on plugin architecture and retrieval quality while AstaBrief bets on a fine-tuned model absorbing the pipeline — the two poles of the same design space.
- Nuances [[Open Source AI Gap Map (Willison)]]: a concrete instance of the "open models" category in the map, with the rare addition of the *training data* also being released — reproduction requires weights plus data plus pipeline, and Ai2 shipped all three.
- Complicates [[Three Kinds of Agentic Search]]: Turnbull's model-centric strategy (fine-tune the LLM to use retrieval efficiently) validated by a production system — but only after the retrieval and harness (ScholarQA) already existed to generate the training data, so the strategies are sequential, not competing.

---
*Sources: [[raw/astabrief]], [[summary/astabrief]]*
*Last updated: 2026-10-03*
