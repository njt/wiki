---
url: https://huggingface.co/shisa-ai/shisa-de-1
date_fetched: 2026-09-22
---

### Instructions to use shisa-ai/shisa-de-1 with libraries, inference providers, notebooks, and local apps. Follow these links to get started.

- Libraries
-  Transformers How to use shisa-ai/shisa-de-1 with Transformers: `# Use a pipeline as a high-level helper from transformers import pipeline pipe = pipeline("text-classification", model="shisa-ai/shisa-de-1")``# Load model directly from transformers import AutoProcessor, AutoModelForMultimodalLM processor = AutoProcessor.from_pretrained("shisa-ai/shisa-de-1") model = AutoModelForMultimodalLM.from_pretrained("shisa-ai/shisa-de-1", device_map="auto")`
- Notebooks
- Google Colab
- Kaggle

# Shisa DE-1

Shisa DE-1 (Decision Engine) is a general-purpose "decision model". It takes one state and a list of named questions and returns one typed answer per question. Instead of an answer text, each answer is read from the next-token distribution at the answer boundary, and restricted to the option letters.

This model was trained from `google/gemma-4-26B-A4B-it` with softmax cross-entropy
over the restricted option-letter distribution at the answer boundary. The loss
is the deployment readout: the next-token logits are restricted to the K option
letter rows and the cross-entropy targets the correct letter.

The vision encoder is present in the weights and was untouched by training.

Train accuracy rose 0.888 → 0.936 → 0.968 across the three epochs.

## Model details

| Field | Value | 
|---|---|
| Model id | `shisa-ai/shisa-de-1` | 
| Developed by | Shisa AI | 
| Model type | Causal language model; no task head is added to the graph | 
| Base model | `google/gemma-4-26B-A4B-it` | 
| Architecture | Gemma 4 mixture of experts: 30 layers, 128 experts with 8 active plus 1 shared, 2,816 hidden size, 262,144-token vocabulary, 256K context | 
| Parameters | 25.2B total and 3.8B active per the base model card | 
| Precision | bfloat16 | 
| Size on disk | 51,612,009,332 bytes (48.1 GiB) in 2 safetensors shards, 1,013 tensors | 
| License | Apache-2.0 | 

## Serving

### Readout contract

Send one request per question. Render the state and the question with the
option list, then read `logprobs` at the answer position with `max_tokens=1`.
Restrict the letter logits to the K valid option letters (`A` upward) and take
the argmax for a choice answer or the softmax over those rows for a `noul`
probability. The model emits no answer string, and its unrestricted next-token
distribution is not part of the contract.

**Example.** One question is one request. Render the prompt with the model's
chat template, then read the logprobs of the first generated token:

```
curl http://localhost:8000/v1/completions \
  -H 'Content-Type: application/json' \
  -d '{"model": "shisa-de-1", "prompt": "<rendered prompt>", "max_tokens": 1, "temperature": 0, "logprobs": 20}'
```
The measured scaffold sends a system instruction and a JSON payload in the user message, then appends the model's generation prompt:

```
<bos><|turn>system
Apply the supplied criterion to the supplied evidence. Choose exactly one listed option. Respond with only its uppercase letter, with no explanation or reasoning.<turn|>
<|turn>user
{"evidence": "From: billing@acme-support.example\nSubject: Your invoice is overdue\n\nWe could not charge your card. Reply with your full card number, expiry, and CVV so we can release the pending refund.", "criterion": "What is this message trying to obtain?", "options": [{"letter": "A", "description": "payment card details"}, {"letter": "B", "description": "a support callback"}, {"letter": "C", "description": "nothing; it is a routine notice"}]}<turn|>
<|turn>model
<|channel>thought
<channel|>
```
That prompt is 161 tokens. The response (truncated) for its three options is:

```
{
  "choices": [{"text": "A", "logprobs": {"top_logprobs": [
    {"A": -0.0056, "B": -6.1306, "D": -7.3806, "C": -7.5056, "-": -7.5056, "1": -9.3806}
  ]}}],
  "usage": {"prompt_tokens": 161, "completion_tokens": 1, "total_tokens": 162}
}
```
Restrict the candidate rows to the option letters and normalize:

| Letter | logprob | probability | 
|---|---|---|
| A | -0.0056 | 0.9973 | 
| B | -6.1306 | 0.0022 | 
| C | -7.5056 | 0.0006 | 

The answer is `A` at 0.9973. Tokens outside the option list are discarded even
when they outrank a valid option, as `D` does here. `choices[0].text` is the
sampled token, shown for debugging only; the contract reads the distribution,
not the sample.

A `noul` question uses the option pair Yes/No, and the normalized probability of
the Yes row is the reported `noul` value. Each option letter must be a single
token, and `prompt + letter` must tokenize as the prompt tokens plus exactly
that letter's token. If a letter is missing from `top_logprobs`, repeat the
request with `prompt_logprobs: 0` and the letter appended to the prompt, and
read the last entry.

### Launch

This model can be served via mainline vLLM:

```
vllm serve shisa-ai/shisa-de-1
```
On 2 x NVIDIA H20-3e (TP2), fraud-detect suite (59 cases, 295 questions, all splits):

| Metric | Value | 
|---|---|
| Throughput plateau | 596-684 req/s from c=64 to c=512 | 
| Peak | 683.9 req/s at c=64 | 
| Throughput at c=256 | 668.8 req/s, p50 147 ms, p95 224 ms | 
| Memory per GPU | 129,429 MiB | 

Single-sweep latency at a median 201 input tokens per case:

| Concurrency | req/s | p50 | p95 | p99 | 
|---|---|---|---|---|
| 1 | 82.6 | 12 ms | 14 ms | 15 ms | 
| 4 | 161.7 | 25 ms | 28 ms | 30 ms | 
| 16 | 405.2 | 38 ms | 51 ms | 54 ms | 
| 64 | 683.9 | 85 ms | 104 ms | 270 ms | 
| 256 | 668.8 | 147 ms | 224 ms | 265 ms | 
| 512 | 659.5 | 135 ms | 169 ms | 196 ms | 

## Training

### Data

The base Gemma 4 model is already quite strong, but we trained on 3,632 training examples for specific use cases, including compaction, fraud detection, anti-slop, and safety guardrails.

Training used softmax cross-entropy restricted to the option-letter rows at the answer boundary, so the training objective is the serving readout. A dot-product head over frozen hidden states was also trained and scored lower on the sealed compaction test: 0.837 AUROC at its best epoch against 0.8902 for this model.

## Evaluation

The quality harness ran 29 suites against this checkpoint, plus option-order-reversed variants of 13 of them.

While even much smaller classifier models can perform admirably at many decision-making tasks, our testing showed that larger LLMs provide better generalization and can still run within real-time performance windows.

Latencies should be taken as broad guidance. While all models were measured on the same-sized runs, some smaller models were run at TP1, while DeepSeek-V4.1-Flash requires TP4, and Jev is a hosted solution and run remotely and includes network latency to our data center.

| Model | Params | fraud-detect | ja-routing | antislop | guardrails | ag_news | p50 ms | Verdict | 
|---|---|---|---|---|---|---|---|---|
| Shisa DE-1 | 26B-A4B / 4B act | 0.999 | 0.976 | 0.997 | 0.917 | 0.895 | 20 | selected | 
| Gemma 4 (base) | 26B-A4B / 4B act | 0.985 | 0.976 | 0.973 | 0.833 | 0.895 | 59 | training base | 
| DeepSeek-V4.1-Flash | 552B-P8B-D16B | 0.995 | 1.000 | 1.000 | 0.750 | 0.905 | 161 | separate hardware class (TP4) | 
| Qwen3.8-Flash-Next | 125B-A6B | 0.983 | 0.976 | 0.970 | 0.583 | 0.910 | 61 | stronger reasoning, loses Japanese | 
| Nimble-9B | 9B | 0.950 | 0.927 | 0.919 | 0.583 | 0.915 | 153 | trained on other decision families | 
| kev-4B (trained) | 4B | 0.857 | 0.750 | 0.871 | 0.500 | 0.925 | 53 | wins only its trained families | 
| GLiFormer-large | 576M | 0.777 | 0.667 | 0.734 | 0.500 | 0.765 | 32 | |
| BGE-M3 zero-shot | 568M | 0.670 | 0.833 | 0.682 | 0.500 | 0.920 | 9 | |
| Laya | 421M | 0.607 | 0.610 | 0.647 | 0.417 | 0.960 | 16 | Japanese rows collapse | 
| GLiNER2.5-base | 194M | 0.732 | 0.750 | 0.804 | 0.500 | 0.705 | 37 | |
| ModernCE-base | 150M | 0.596 | 0.317 | 0.676 | 0.417 | 0.905 | 9 | English NLI cross-encoder | 
| Jev (hosted) | undisclosed | 0.982 | 1.000 | 0.970 | 0.583 | 0.900 | 223 | baseline this model replaces | 

### All suites measured on this checkpoint

Metric per suite is the suite's primary metric, and all three models ran the same questions.

| Suite | Metric | Shisa DE-1 | Jev | Gemma 4 base | 
|---|---|---|---|---|
| fraud-detect | AUROC | 0.9994 | 0.9823 | 0.9853 | 
| compaction | AUROC | 0.8587 | 0.9228 | 0.8754 | 
| guardrails | acc | 0.9167 | 0.5833 | 0.8333 | 
| antislop | AUROC | 0.9968 | 0.9704 | 0.9725 | 
| ja-routing | acc | 0.9756 | 1.0000 | 0.9756 | 
| ja-register | AUROC | 0.9970 | 0.9820 | 0.9995 | 
| ja-leakage | AUROC | 0.9040 | 0.8963 | 0.8956 | 
| ja-indirect | AUROC | 0.6871 | 0.8440 | 0.8955 | 
| ja-nuance | AUROC | 0.9434 | 0.9208 | 0.9455 | 
| classifier-benchmark | acc | 1.0000 | 1.0000 | 1.0000 | 
| openjev-authored144 | acc | 0.9444 | 0.9722 | 0.9514 | 
| simplejev-noul-probe | AUROC | 1.0000 | 1.0000 | 1.0000 | 
| jdb-probe-counting | acc | 0.6533 | 0.8667 | 0.7333 | 
| jdb-probe-fresh_math | acc | 0.5333 | 0.7167 | 0.6917 | 
| jdb-probe-two_hop | acc | 0.9800 | 1.0000 | 0.9733 | 
| jdb-probe-date_compare | AUROC | 0.9921 | 1.0000 | 0.9925 | 
| jdb-probe-double_negation | AUROC | 1.0000 | 1.0000 | 1.0000 | 
| jdb-probe-number_compare | AUROC | 0.9998 | 1.0000 | 0.9995 | 
| kev-decision-v1 * | acc | 0.6925 | 0.7951 | 0.6944 | 
| kev-transfer-v1 | acc | 0.4914 | 0.8103 | 0.5075 | 
| jeff8-ag_news | acc | 0.8950 | 0.9000 | 0.8950 | 
| jeff8-emotion | acc | 0.4350 | 0.4650 | 0.4400 | 
| jeff8-boolq | AUROC | 0.9502 | 0.9512 | 0.9402 | 
| jeff8-irony | AUROC | 0.9524 | 0.9563 | 0.9699 | 
| jeff8-sms_spam | AUROC | 0.9871 | 0.9948 | 0.9891 | 
| jeff8-sst2 | AUROC | 0.9931 | 0.9947 | 0.9942 | 
| jeff8-amazon | MAE | 0.415 | 0.445 | 0.470 | 
| jeff8-sst5 | MAE | 0.515 | 0.490 | 0.460 | 
| contract-smoke | all types | 0.9231 | 1.0000 | 0.9231 | 

* `kev-decision-v1` is a partial run for this checkpoint and for Gemma 4 base:
272 of its 776 choice questions are Banking77 `intent` questions with 77 options,
which exceed the 26 single-token letter slots this readout restricts to. Hosted
Jev answered all 776.

### Choice accuracy and option-order sensitivity

Reversal rewrites each choice question's option order and leaves gold keyed by option id, so a position-invariant model scores identically. Deltas are reversed minus original. The original column is also this model's choice accuracy on those suites.

| Suite | Original | Reversed | Delta | 
|---|---|---|---|
| classifier-benchmark | 1.000 | 1.000 | +0.000 | 
| fraud-detect | 0.983 | 0.975 | -0.008 | 
| guardrails | 0.917 | 0.917 | +0.000 | 
| ja-routing | 0.976 | 0.951 | -0.024 | 
| jdb-probe-counting | 0.653 | 0.500 | -0.153 | 
| jdb-probe-fresh_math | 0.533 | 0.475 | -0.058 | 
| jdb-probe-two_hop | 0.980 | 0.967 | -0.013 | 
| jeff8-ag_news | 0.895 | 0.855 | -0.040 | 
| jeff8-emotion | 0.435 | 0.450 | +0.015 | 
| kev-decision-v1 | 0.693 | 0.675 | -0.018 | 
| kev-transfer-v1 | 0.491 | 0.453 | -0.039 | 
| openjev-authored144 | 0.944 | 0.931 | -0.014 | 
| simplejev-noul-probe | 1.000 | 1.000 | +0.000 | 

Mean absolute shift 0.029. The 11-option counting probe is the worst case, and the chosen-count distribution moves with the presentation: on identical cases and gold, the model picks count 3 on 22 cases ascending and 40 descending. The readout is not scoring the option text alone.

### Calibration

A post-hoc temperature is fitted on this checkpoint's dev-split rows by NLL
grid search, separately per answer kind because `noul` and choice calibrate
differently: T_noul = 1.69 and T_choice = 1.90 (T_global = 1.84). On the test
split (n=7,446), noul ECE moves from 0.054 to 0.042 and choice ECE from 0.138 to
0.046. Accuracy is unchanged, because tempering does not move the argmax.
Leave-one-suite-out fits span 1.76 to 1.89, so one temperature transfers across
suites. Refit on your own rows before treating the `noul` probabilities as
confidence.

## Limitations

- **Held-out decision families are unsolved.**- `kev-transfer-v1`scores 0.491 against hosted Jev's 0.810. A prompt-scaffold A/B moved nothing, so this is a readout limit rather than a rendering artifact.
- **Arithmetic and counting stay in code.**jdb-probe-counting 0.653 and jdb-probe-fresh_math 0.533 on these weights, and the failure class appears at every model size tested.
- **Option order changes answers.**Reversal costs 15 points on the 11-option counting probe and 4 points on- `jeff8-ag_news`.
- **No generation quality is claimed.**Only the restricted-letter readout was trained and measured. Open-ended generation, chat, and tool use are unsupported by the evidence here, and the vision and audio paths were not evaluated at all.

- Downloads last month
- -
