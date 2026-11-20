---
url: https://huggingface.co/RadixArk/DaisyRose-27B
date_fetched: 2026-11-20
---

### Instructions to use RadixArk/DaisyRose-27B with libraries, inference providers, notebooks, and local apps. Follow these links to get started.

- Libraries
-  Transformers How to use RadixArk/DaisyRose-27B with Transformers: `# Use a pipeline as a high-level helper from transformers import pipeline pipe = pipeline("image-text-to-text", model="RadixArk/DaisyRose-27B") messages = [ { "role": "user", "content": [ {"type": "image", "url": "https://huggingface.co/datasets/huggingface/documentation-images/resolve/main/p-blog/candy.JPG"}, {"type": "text", "text": "What animal is on the candy?"} ] }, ] pipe(text=messages)``# pip install -U transformers accelerate # Load model directly from transformers import AutoProcessor, AutoModelForMultimodalLM processor = AutoProcessor.from_pretrained("RadixArk/DaisyRose-27B") model = AutoModelForMultimodalLM.from_pretrained("RadixArk/DaisyRose-27B", device_map="auto") messages = [ { "role": "user", "content": [ {"type": "image", "url": "https://huggingface.co/datasets/huggingface/documentation-images/resolve/main/p-blog/candy.JPG"}, {"type": "text", "text": "What animal is on the candy?"} ] }, ] inputs = processor.apply_chat_template( messages, add_generation_prompt=True, tokenize=True, return_dict=True, return_tensors="pt", ).to(model.device) outputs = model.generate(**inputs, max_new_tokens=256) print(processor.decode(outputs[0][inputs["input_ids"].shape[-1]:]))`
- Notebooks
- Google Colab
- Kaggle
- Local Apps Settings
-  vLLM  How to use RadixArk/DaisyRose-27B with vLLM: ##### Install from pip and serve model`# Install vLLM from pip: pip install vllm # Start the vLLM server: vllm serve "RadixArk/DaisyRose-27B" # Call the server using curl (OpenAI-compatible API): curl -X POST "http://localhost:8000/v1/chat/completions" \ -H "Content-Type: application/json" \ --data '{ "model": "RadixArk/DaisyRose-27B", "messages": [ { "role": "user", "content": [ { "type": "text", "text": "Describe this image in one sentence." }, { "type": "image_url", "image_url": { "url": "https://cdn.britannica.com/61/93061-050-99147DCE/Statue-of-Liberty-Island-New-York-Bay.jpg" } } ] } ] }'`##### Use Dockerdocker model run hf.co/RadixArk/DaisyRose-27B 
-  SGLang  How to use RadixArk/DaisyRose-27B with SGLang: ##### Install from pip and serve model`# Install SGLang from pip: pip install sglang # Start the SGLang server: python3 -m sglang.launch_server \ --model-path "RadixArk/DaisyRose-27B" \ --host 0.0.0.0 \ --port 30000 # Call the server using curl (OpenAI-compatible API): curl -X POST "http://localhost:30000/v1/chat/completions" \ -H "Content-Type: application/json" \ --data '{ "model": "RadixArk/DaisyRose-27B", "messages": [ { "role": "user", "content": [ { "type": "text", "text": "Describe this image in one sentence." }, { "type": "image_url", "image_url": { "url": "https://cdn.britannica.com/61/93061-050-99147DCE/Statue-of-Liberty-Island-New-York-Bay.jpg" } } ] } ] }'`##### Use Docker images`docker run --gpus all \ --shm-size 32g \ -p 30000:30000 \ -v ~/.cache/huggingface:/root/.cache/huggingface \ --env "HF_TOKEN=<secret>" \ --ipc=host \ lmsysorg/sglang:latest \ python3 -m sglang.launch_server \ --model-path "RadixArk/DaisyRose-27B" \ --host 0.0.0.0 \ --port 30000 # Call the server using curl (OpenAI-compatible API): curl -X POST "http://localhost:30000/v1/chat/completions" \ -H "Content-Type: application/json" \ --data '{ "model": "RadixArk/DaisyRose-27B", "messages": [ { "role": "user", "content": [ { "type": "text", "text": "Describe this image in one sentence." }, { "type": "image_url", "image_url": { "url": "https://cdn.britannica.com/61/93061-050-99147DCE/Statue-of-Liberty-Island-New-York-Bay.jpg" } } ] } ] }'`
-  Docker Model Runner  How to use RadixArk/DaisyRose-27B with Docker Model Runner: docker model run hf.co/RadixArk/DaisyRose-27B 

# DaisyRose-27B

BF16 | FP8 | NVFP4 | GGUF | DSpark draft | Base model | Blog

This repository holds the **BF16** weights of DaisyRose-27B.

## 1. General Information

DaisyRose-27B is RadixArk's first model: a multimodal reasoning model built on Qwen3.8-27B with reinforcement learning only, without any supervised fine-tuning or distillation. It was trained on about 1,200 curated long-horizon agent tasks, with sessions of up to 1M tokens, for 18 days on 144 H200 GPUs. Terminal Bench 4.1 pass@1 rises from 5.6 to 12.0, and the model also improves on most of the other coding, reasoning and instruction-following benchmarks below.

DaisyRose-27B keeps the base model's architecture, tokenizer and chat template, along with its image understanding and 256K-token context, so it is a drop-in replacement for Qwen3.8-27B in existing inference and post-training stacks. Weights are released under Apache-2.0 in BF16, FP8, and NVFP4.

## 2. Evaluations

**DaisyRose-27B against its base model, open models of similar or larger size, and GPT-6 Luna**

| Benchmark | DaisyRose-27B | Qwen3.8-27B | Muse Glimmer-30B | Qwen3.6-35B-A3B | MiMo V2.6 Flash | Inkling 2 Small | Beam 1.0 | Muse Spark Open | GPT-6 Luna | 
|---|---|---|---|---|---|---|---|---|---|
| Activated parameters | 27B | 27B | 30B | 3B | 15B | 17B | 23B | 42B | – | 
| Total parameters | 27B | 27B | 30B | 35B | 309B | 594B | 501B | 816B | – | 
| Terminal Bench 4.1 | 12.0 | 5.6 | 0.5 | 0.0 | 16.1 | 13.0 | 6.4 | 6.6 | 15.2 | 
| Terminal Bench 2.1 | 75.5 | 73.0 | 51.7 | 44.9 | 87.6 | 81.3 | 80.1 | 76.4 | 73.0 | 
| DeepSWE v1.1 | 46.2 | 42.2 | 4.4 | 0.0 | 67.9 | 72.1 | 43.3 | 60.0 | 66.6 | 
| SWE Bench Pro v2 | 94.7 | 91.3 | 80.4 | 73.3 | – | 97.7 | – | 95.0 | – | 
| HLE (text only) | 34.3 | 33.9 | 22.0 | 22.2 | 35.1 | 44.1 | 34.7 | 43.9 | 38.5 | 
| IFBench | 82.4 | 79.5 | 77.0 | 64.4 | – | – | 79.6 | – | – | 
| GPQA Diamond | 90.7 | 89.2 | 83.5 | 86.0 | 88.7 | 93.0 | 90.6 | 89.0 | 90.5 | 

## How we evaluated

**Terminal Bench 4.1:** Averaged over 8 repetitions per task, internal mini-swe-agent harness with summarization enabled.**Terminal Bench 2.1:** Averaged over 5 repetitions per task, Terminus 2 harness.**DeepSWE v1.1:** Averaged over 8 repetitions per task, mini-swe-agent harness.**SWE Bench Pro v2:** Averaged over 5 repetitions per task, mini-swe-agent harness. Qwen3.8-27B score from Table 3 of the SWE Bench Pro v2 paper (full 642-task set, mini-swe-agent).**HLE (text only):** Averaged over 5 repetitions per problem, GPT-6 Luna grader, no tools.**IFBench:** Averaged over 4 repetitions per prompt (300 prompts), official allenai/IFBench scorer, prompt-level loose accuracy on the final response (thinking excluded).**GPQA Diamond:** Averaged over 8 repetitions per question (198 questions), simple-evals multiple-choice template and answer extraction, answer order reshuffled each repetition.

All DaisyRose-27B runs use the base model's recommended thinking-mode settings: temperature 1.0, top_p 0.95, top_k 20, min_p 0, reasoning effort xhigh (the template default), no max_tokens cap within the 256K context. DaisyRose-27B scores are the higher of our BF16 and FP8 runs, which differ by less than one point on every benchmark where we ran both. Other models' numbers are as reported by their developers: see the sources below.

## Sources

Terminal Bench 4.1: MiMo V2.6 Flash, Inkling 2 Small and Beam 1.0 from the official leaderboard (mini-swe-agent); GPT-6 Luna from the same leaderboard (Codex, max effort). The other models have no 4.1 entry, so Terminal Bench 4.0 scores are shown instead: Muse Spark Open from Vals.ai (average of 3, mini-swe-agent); Qwen3.8-27B (xhigh), Muse Glimmer-30B and Qwen3.6-35B-A3B from Artificial Analysis (Qwen3.8, Glimmer, Qwen3.6).

Terminal Bench 2.1: model cards for Qwen3.8-27B, Muse Glimmer-30B (Terminus 2), Inkling 2 Small (Terminus 2), Beam 1.0 and Muse Spark Open (Terminus 2). GPT-6 Luna from the Inkling 2 Small model card (Terminus 2). Qwen3.6-35B-A3B from Artificial Analysis (Terminus 2). MiMo V2.6 Flash from the MiMo-V2.6 release (harness not stated).

DeepSWE v1.1: model cards for Qwen3.8-27B, Inkling 2 Small (mini-swe-agent v2.4.6), Beam 1.0 and Muse Spark Open. Muse Glimmer-30B from the Nanbeige4.3-33B model card (mini-swe-agent v2.4.6). Qwen3.6-35B-A3B from the Ornith-1.5-35B-A3B model card (Claude Code). MiMo V2.6 Flash from the MiMo-V2.6 release. GPT-6 Luna from the GPT-6 Sol and Luna release (max effort).

SWE Bench Pro v2 (Full, 642 tasks): Qwen3.8-27B and Muse Glimmer-30B are the Base rows of Table 3 in Eliciting and Mitigating Specification Satisficing in Coding Agents. Qwen3.6-35B-A3B is the base-model column of the Logics-SWE-Qwen3.6-35B-A3B model card (mini-SWE-agent 2.4.6, mean of 3 rounds). Inkling 2 Small from its model card (mini-swe-agent v2.4.6). Muse Spark Open from the Scale SWE-Bench Pro V2 leaderboard. MiMo V2.6 Flash, Beam 1.0 and GPT-6 Luna report only the Hard subset.

HLE (text only): Muse Glimmer-30B from its model card (measured by Artificial Analysis). MiMo V2.6 Flash, Inkling 2 Small and GPT-6 Luna from the Inkling 2 Small model card. Qwen3.8-27B (xhigh), Qwen3.6-35B-A3B (reasoning), Beam 1.0 and Muse Spark Open from Artificial Analysis (Qwen3.8, Qwen3.6, Beam, Muse Spark Open).

IFBench: model cards for Qwen3.8-27B, Muse Glimmer-30B and Beam 1.0. Qwen3.6-35B-A3B from Artificial Analysis.

GPQA Diamond: model cards for Qwen3.8-27B, Muse Glimmer-30B (measured by Artificial Analysis), Qwen3.6-35B-A3B, Beam 1.0 and Muse Spark Open. MiMo V2.6 Flash, Inkling 2 Small and GPT-6 Luna from the Inkling 2 Small model card.

## 3. Getting Started

### Serving with SGLang

We strongly recommend serving DaisyRose-27B with SGLang and DSpark speculative decoding, using the official `lmsysorg/sglang:latest` Docker image (DSpark is included).

The SGLang cookbook for DaisyRose-27B generates the launch command for each GPU and precision. BF16 on a 4-GPU node, one replica per GPU:

```
docker run --gpus all --ipc=host --shm-size 32g -p 30000:30000 \
  -v ~/.cache/huggingface:/root/.cache/huggingface \
  lmsysorg/sglang:latest \
  python3 -m sglang.launch_server \
  --model-path RadixArk/DaisyRose-27B \
  --served-model-name DaisyRose-27B \
  --tp-size 1 --dp-size 4 \
  --context-length 262144 \
  --reasoning-parser qwen3 \
  --tool-call-parser qwen3_coder \
  --trust-remote-code \
  --mem-fraction-static 0.85 \
  --kv-cache-dtype fp8_e4m3 \
  --chunked-prefill-size 2048 \
  --mamba-full-memory-ratio 11.01 \
  --mamba-radix-cache-strategy extra_buffer_lazy \
  --mamba-ssm-dtype float32 \
  --speculative-algorithm DSPARK \
  --speculative-draft-model-path RadixArk/Qwen3.8-27B-DSpark \
  --speculative-draft-attention-backend flashinfer \
  --host 0.0.0.0 --port 30000
```
### Supported hardware

DaisyRose-27B runs on a single GPU, from data-center cards (H200, B200, GB300) to local hardware such as RTX PRO 6000, RTX 5090 and DGX Spark. The weights take about 52 GB in BF16, 29 GB in FP8 and 23 GB in NVFP4; NVFP4 needs a Blackwell GPU. See the SGLang cookbook for DaisyRose-27B for per-GPU launch settings.

### Quickstart

The example uses the `openai` Python client against the server above. The model thinks before answering by default; the thinking comes back in `reasoning_content` and the answer in `content`.

```
from openai import OpenAI
client = OpenAI(base_url="http://localhost:30000/v1", api_key="none")
SAMPLING = dict(temperature=1.0, top_p=0.95, extra_body={"top_k": 20, "min_p": 0.0})
resp = client.chat.completions.create(
    model="DaisyRose-27B",
    messages=[{"role": "user", "content": "How many positive divisors does 360 have?"}],
    **SAMPLING,
)
print(resp.choices[0].message.reasoning_content)  # thinking
print(resp.choices[0].message.content)            # final answer
```
- Use `temperature 1.0`,`top_p 0.95`,`top_k 20`,`min_p 0`in thinking mode.
- Leave `max_tokens`unset for hard problems; long thinking runs to tens of thousands of tokens.
- In multi-turn chats, send earlier turns back with their `reasoning_content`; the chat template keeps it by default.

## 4. Model Properties

| Property | Value | 
|---|---|
| Model type | Multimodal causal language model with a vision encoder; thinking model | 
| Architecture type | Qwen3.8-27B ( `Qwen3_5ForConditionalGeneration`): 64 decoder layers in a 3:1 pattern of linear-attention (Gated DeltaNet) and full-attention layers, hidden size 5,120, a 27-layer vision encoder, and one multi-token-prediction layer | 
| Parameters | 27.8B in total, including a 0.46B vision encoder and a 0.42B multi-token-prediction head | 
| Context length | 262,144 tokens | 
| Numerics support | BF16, FP8, NVFP4 | 
| Input modalities | Text, images | 
| Output modalities | Text | 

## 5. Training

DaisyRose-27B is trained from Qwen3.8-27B with reinforcement learning only. The vision encoder and the multi-token-prediction head are the base model's, unchanged.

**Training data.** We built the tasks with Eigent AI, starting from its large pool of tasks in machine learning, software engineering, science and other fields. About 1,200 made it into the final training set. Each task targets a mistake the base model makes and is tuned until its solve rate is neither too easy nor too hard to learn from. Each is a long job of several interdependent problems: an attempt gets up to 1.5 hours in a sandbox, with more than 100 tool calls and more than 256K output tokens. We reviewed every task together with the model's attempts, and fixed or removed any that could be passed without really solving it.

**Training infrastructure.** Training ran fully asynchronously with Miles, our open-source reinforcement-learning framework, on RadixArk's internal training backend. 128 H200s ran the agent on its tasks, served by SGLang, while 16 more trained on the finished trajectories; after every step the updated weights went back to the 128. Each step learns from a handful of very long sessions, some as long as 1M tokens. The run took 18 days on 144 H200s, about 63,000 GPU-hours. Every attempt ran in a sandbox on our own sandbox service, built on AgentENV.

## 6. Acknowledgements

We thank Eigent AI for building the training data with us, and the Qwen team for Qwen3.8-27B. We also thank the Miles and SGLang communities and other open-source projects, including AgentENV and Harbor.

## 7. License

**GOVERNING DOWNLOAD TERMS:** Use of the model is governed by the Apache 2.0 license. DaisyRose-27B is a derivative of Alibaba's Qwen3.8-27B, which is also licensed under Apache 2.0.

- Downloads last month
- 166
