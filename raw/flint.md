---
url: https://huggingface.co/RecursiveSI/flint
date_fetched: 2026-10-24
---

### Instructions to use RecursiveSI/flint with libraries, inference providers, notebooks, and local apps. Follow these links to get started.

- Libraries
-  Transformers How to use RecursiveSI/flint with Transformers: `# Use a pipeline as a high-level helper from transformers import pipeline pipe = pipeline("text-generation", model="RecursiveSI/flint") messages = [ {"role": "user", "content": "Who are you?"}, ] pipe(messages)``# pip install -U transformers accelerate # Load model directly from transformers import AutoTokenizer, AutoModelForCausalLM tokenizer = AutoTokenizer.from_pretrained("RecursiveSI/flint") model = AutoModelForCausalLM.from_pretrained("RecursiveSI/flint", device_map="auto") messages = [ {"role": "user", "content": "Who are you?"}, ] inputs = tokenizer.apply_chat_template( messages, add_generation_prompt=True, tokenize=True, return_dict=True, return_tensors="pt", ).to(model.device) outputs = model.generate(**inputs, max_new_tokens=256) print(tokenizer.decode(outputs[0][inputs["input_ids"].shape[-1]:]))`
- Notebooks
- Google Colab
- Kaggle
- Local Apps Settings
-  vLLM  How to use RecursiveSI/flint with vLLM: ##### Install from pip and serve model`# Install vLLM from pip: pip install vllm # Start the vLLM server: vllm serve "RecursiveSI/flint" # Call the server using curl (OpenAI-compatible API): curl -X POST "http://localhost:8000/v1/chat/completions" \ -H "Content-Type: application/json" \ --data '{ "model": "RecursiveSI/flint", "messages": [ { "role": "user", "content": "What is the capital of France?" } ] }'`##### Use Dockerdocker model run hf.co/RecursiveSI/flint 
-  SGLang  How to use RecursiveSI/flint with SGLang: ##### Install from pip and serve model`# Install SGLang from pip: pip install sglang # Start the SGLang server: python3 -m sglang.launch_server \ --model-path "RecursiveSI/flint" \ --host 0.0.0.0 \ --port 30000 # Call the server using curl (OpenAI-compatible API): curl -X POST "http://localhost:30000/v1/chat/completions" \ -H "Content-Type: application/json" \ --data '{ "model": "RecursiveSI/flint", "messages": [ { "role": "user", "content": "What is the capital of France?" } ] }'`##### Use Docker images`docker run --gpus all \ --shm-size 32g \ -p 30000:30000 \ -v ~/.cache/huggingface:/root/.cache/huggingface \ --env "HF_TOKEN=<secret>" \ --ipc=host \ lmsysorg/sglang:latest \ python3 -m sglang.launch_server \ --model-path "RecursiveSI/flint" \ --host 0.0.0.0 \ --port 30000 # Call the server using curl (OpenAI-compatible API): curl -X POST "http://localhost:30000/v1/chat/completions" \ -H "Content-Type: application/json" \ --data '{ "model": "RecursiveSI/flint", "messages": [ { "role": "user", "content": "What is the capital of France?" } ] }'`
-  Docker Model Runner  How to use RecursiveSI/flint with Docker Model Runner: docker model run hf.co/RecursiveSI/flint 


# Recursive Flint

Blog | Hugging Face | Local engine | Documentation

Flint is Recursive’s mixture-of-experts language model for coding agents, with 39.8 billion total parameters and approximately 2.5 billion active per token. It is designed to investigate codebases, fix bugs, implement features, and write tests.

Flint supports text input and output, reasoning, and multi-turn tool use. In our release evaluations, it scores 54.9% on Terminal-Bench 2.1, 89.7% on SWE-bench Pro v2, and 74.2% on SWE-bench Verified.

This repository contains the BF16 checkpoint, model configuration, tokenizer, chat template, and an SGLang serving kit. Quantized variants are prepared separately for NVIDIA and Apple Silicon runtimes.

## Evaluation

Flint results use temperature 1.0, top-p 0.95, and top-k 20. Terminal-Bench uses Terminus 2; SWE-bench uses mini-swe-agent. Agentic scores report average pass@1 over at least five attempts. † marks scores measured by Recursive; other comparison scores are developer-reported.

Evaluation setups vary across sources. Flint’s Terminal-Bench score counts solutions obtained through web retrieval as unsuccessful; developer-reported scores may not apply the same adjustment.

## Model specifications

| Property | Flint | 
|---|---|
| Developer | Recursive | 
| Architecture | Decoder-only mixture-of-experts Transformer | 
| Total parameters | 39.8B | 
| Active parameters per token | Approximately 2.5B, including the output head¹ | 
| Layers | 49: 2 dense layers followed by 47 MoE layers | 
| Attention | 35 sliding-window layers and 14 full-attention layers | 
| Sliding window | 513 tokens: the current token and 512 preceding tokens | 
| Attention heads | 32 query heads and 4 key/value heads; head dimension 128 | 
| Attention normalization | Per-head QK RMSNorm and learned per-head attention sinks | 
| Position encoding | RoPE on sliding-window layers; no positional encoding on full-attention layers | 
| Experts per MoE layer | 256 routed experts, with 6 selected per token, plus an always-active shared expert | 
| Documented context configuration | 262,144 tokens, including prompt and generated output | 
| Vocabulary | 150,144 tokens | 
| Input / output | Text / text | 
| Stated knowledge cutoff | June 2026 | 

¹ The active-parameter convention includes the output head and excludes the full input embedding matrix. The active body alone is approximately 2.2B parameters. “40B-A2.5B” is the rounded model-size designation.

## Getting started

Flint runs on one B300 or H200 GPU per replica with the SGLang kit in `sglang/`. Transformers modeling code is not included yet.

```
pip install -U huggingface_hub openai
hf download RecursiveSI/flint --local-dir flint && cd flint   # about 80 GB
docker build -t flint-sglang sglang
docker run -d --name flint --gpus '"device=0"' --ipc=host -p 127.0.0.1:30000:30000 \
  -v "$PWD:/model" -e MODEL=/model flint-sglang bash /model/sglang/serve_b300.sh   # H200: serve_h200.sh
until curl -sf localhost:30000/health_generate >/dev/null; do sleep 15; done      # progress: docker logs -f flint
python3 sglang/example_client.py --base-url http://localhost:30000/v1              # reasoning and a tool call
```
```
from openai import OpenAI
client = OpenAI(base_url="http://localhost:30000/v1", api_key="unused")
response = client.chat.completions.create(
    model="flint",
    messages=[{"role": "user", "content": "Write a Python function that merges overlapping intervals."}],
    max_tokens=32768,
)
print(response.choices[0].message.content)
```
`sglang/README.md` covers requirements, serve-script options, multi-GPU deployments, and API details.

### Recommended generation settings

| Setting | Value | 
|---|---|
| Temperature | 1.0 | 
| Top-p / top-k | 0.95 / 20 | 
| Output budget for hard problems | At least 32768 tokens, including reasoning | 

The server applies these sampling settings when a request omits them. Flint supports OpenAI-compatible tool calls. Use the checkpoint’s chat template and preserve returned `reasoning_content` in conversation history. Prompt and output share the context window.

## Training

Flint was trained from scratch through pretraining, mid-training, supervised fine-tuning, and reinforcement learning. Training includes text and code, instruction following, and agentic tasks.

## Limitations and responsible use

Flint’s supervised fine-tuning included safety-oriented examples, but the model can still produce incorrect or harmful content, faulty code, or inappropriate tool calls. Test generated code and review consequential changes before applying them. Tool-using applications should enforce their own permissions and execution boundaries; structured tool-call grammar does not prevent the model from choosing an unsafe or incorrect action.

Long reasoning can increase latency and exhaust the output budget before an answer is complete. The context configuration is a capacity limit, not a guarantee of uniform accuracy across all sequence lengths or tasks. Results can vary with checkpoint precision and inference runtime; evaluate the specific checkpoint and runtime you intend to deploy.

## License and reporting

Flint is released under the Apache 2.0 license. Hosted API terms and data practices are separate from the weight license.

For model and serving issues, contact support@recursive.com and include the checkpoint revision, runtime version, hardware, launch command, and a minimal reproduction. Report safety concerns or suspected misuse to safety@recursive.com, and security vulnerabilities to security@recursive.com.

- Downloads last month
- 1,904
