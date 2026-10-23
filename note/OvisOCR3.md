# OvisOCR3 (ATH-MaaS)

OvisOCR3 is an open-weight 0.8B multimodal model from ATH-MaaS that unifies document parsing, general OCR, key information extraction, document figure captioning, and cross-page table reconstruction into one checkpoint, with task selection via prompt files instead of separate deployments. It claims state-of-the-art results among comparable-size models across all five task domains, with a particular edge on cross-page table reconstruction.

---

The interesting structural idea is that a single small model covers five document tasks through **task-specific prompts that define output contracts**: a prompt file pins the exact output shape (structured JSON with `bbox`/`category`/`content`, HTML tables, LaTeX in `$$...$$`, Mermaid for flowcharts), and the model fills it. That's specification-as-interface applied to a vision model — the same pattern the wiki has documented in coding agents, where a written contract between caller and model substitutes for bespoke tooling.

Built on Qwen3.5-0.8B, trained in four stages (continued pretraining, mid-training, on-policy distillation, reinforcement learning). The benchmark tables show it beating substantially larger models on several document metrics — OCRBench v2 EN 65.9 vs 48.1 for Qwen3.5-2B — and a remarkable 97.07 relation F1 on CrossTabBench against Qwen3.8-Max's 94.47, via a two-stage pipeline: parse each page, then reconstruct continued tables from adjacent page pairs, recursively merged for longer spans.

The card's engineering honesty is its best feature. It ships actual postprocessing code — a `_clean_truncated_repeats` function that detects degenerate repeated tails in generations — and admits the model's failure modes plainly:

> "Adherence to the template is prompted, not enforced by constrained decoding in this example; validate generated JSON before downstream use."

> "Treat generated HTML and Mermaid as untrusted content. Sanitize or isolate them before rendering in a browser."

That last line is notable: the model card itself carries the security advice most teams learn the hard way — document-pipeline output is attacker-controlled input if documents come from outside.

> "These are usage settings, not benchmark reproduction settings."

A rare explicit firewall between serving documentation and benchmark claims, resisting the usual model-card drift where demo code gets read as the evaluation protocol.

**Key themes:** #concept (prompt-as-output-contract) #tool #pattern (recursive pairwise merging for long-table reconstruction)

## Analysis

Three things worth flagging. First, the specialization thesis: a 0.8B model trained narrowly beats general-purpose multimodal models many times its size on document tasks, echoing the wiki's recurring finding that small targeted models are the economically rational deployment for well-bounded extraction work. Second, the output-handling burden is real — the card spends more lines on cleanup, validation, and crop-extraction code than on model invocation, an honest ratio that most document-AI marketing inverts. Third, the "prompted, not enforced" admission matters: the structured-output reliability story of small models still rests on the caller's validation layer, not the decoder.

The cross-page table pipeline is quietly the most interesting architecture here: the model itself never sees more than two pages at once; the *system* achieves multi-document reasoning through a deterministic merge loop that re-renders intermediate state. That's an agent-loop pattern applied to OCR.

## Relations

This source strengthens [[Book OCR Project Report — Structured Workflow Runtime and Manual PDF Repair]] — it documents the same class of document-pipeline engineering (parsing, layout, postprocessing, crop management) but from the model vendor's side, with postprocessing code the project would otherwise have to write itself.

It nuances [[Local Qwen Is Not a Worse Opus]] — OvisOCR3 is a Qwen3.5 derivative whose benchmark story is exactly that thesis in miniature: a small local Qwen-family model, specialized, beating larger general models on its niche.

It complicates [[How Far Behind Are Open Models]] — an Apache-2.0, 0.8B model claiming SOTA on five document benchmarks is evidence that in narrow domains open models aren't lagging closed ones at all; the gap rhetoric only makes sense at the frontier of generality.

---
*Sources: [[raw/ovisocr3]], [[summary/ovisocr3]]*
*Last updated: 2026-10-24*
